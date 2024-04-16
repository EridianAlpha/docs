# 🔥 Prometheus

## UFW Config

```bash
sudo ufw allow 9090 comment 'Allow Prometheus UI in'
sudo ufw allow 9093 comment 'Allow Alertmanager UI in'
```

## Docker Config

### Create Directories

{% code fullWidth="false" %}
```bash
mkdir ~/alerting
mkdir ~/alerting/prometheus
mkdir ~/alerting/alertmanager
```
{% endcode %}

{% hint style="info" %}
Since prometheus doesn't support direct variable replacement in the .yml configuration, I'm using a template and script to run when the docker image starts to create the prometheus.yml file dynamically.

This is useful so it uses the correct ports from the execution and beacon config files automatically.
{% endhint %}

### Prometheus Entrypoint Script

```bash
vim ~/alerting/prometheus/entrypoint.sh
```

```bash
#!/bin/sh

# Source environment variables
source /etc/default/execution-variables.env
source /etc/default/beacon-variables.env
export $(cut -d= -f1 /etc/default/execution-variables.env)
export $(cut -d= -f1 /etc/default/beacon-variables.env)

# Output file
OUTPUT="/tmp/prometheus.yml"

# Start with an empty output file
: > "$OUTPUT"

# Process the template file with awk to replace environment variables
awk '{
    while (match($0, /\$\{[^}]+\}/)) {
        varname = substr($0, RSTART + 2, RLENGTH - 3);
        value = ENVIRON[varname];
        if (value == "") value = "UNDEFINED";
        $0 = substr($0, 1, RSTART - 1) value substr($0, RSTART + RLENGTH);
    }
    print;
}' "/etc/prometheus/prometheus.yml.template" > "$OUTPUT"

# Continue with Prometheus startup
exec /bin/prometheus "$@"
```

```bash
# Make the scrip executable by everyone, not just the current user
chmod +x ~/alerting/prometheus/entrypoint.sh
```

### Create Docker Compose

```bash
vim ~/alerting/docker-compose.yml
```

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./prometheus:/etc/prometheus
      - ./prometheus/alert.rules.yml:/etc/prometheus/alert.rules.yml
      - /etc/default/execution-variables.env:/etc/default/execution-variables.env
      - /etc/default/beacon-variables.env:/etc/default/beacon-variables.env
    entrypoint: ["/etc/prometheus/entrypoint.sh"]
    command:
      - '--config.file=/tmp/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'

  alertmanager:
    image: prom/alertmanager:latest
    depends_on:
      - prometheus
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./alertmanager:/etc/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
```

## Prometheus Config

{% hint style="info" %}
Since each client has a different URL path for metrics, and I want a unified endpoint for Prometheus to use, configure an NGINX server to redirect requests to a single endpoint.

It will try each possible endpoint until it finds the actively running client, and if it doesn't find any, it will assume that it is down.
{% endhint %}

### Install NGINX

```bash
sudo apt-get update
sudo apt-get install -y nginx
```

### NGINX Config Script

{% hint style="info" %}
Edit this script to add additional clients metrics paths.
{% endhint %}

```bash
vim ~/nginx_config_script.sh
```

{% code fullWidth="true" %}
```bash
#!/bin/bash

source /etc/default/execution-variables.env
source /etc/default/beacon-variables.env

EXECUTION_METRICS_FULL_URL=""

# Check if Geth service is running
status_code=$(curl -o /dev/null -s -w "%{http_code}" http://localhost:${EXECUTION_METRICS_PORT}/debug/metrics/prometheus)
if [ "$status_code" = "200" ]; then
  EXECUTION_METRICS_FULL_URL=http://localhost:${EXECUTION_METRICS_PORT}/debug/metrics/prometheus
fi

# Check if Besu service is running
status_code=$(curl -o /dev/null -s -w "%{http_code}" http://localhost:${EXECUTION_METRICS_PORT}/metrics)
if [ "$status_code" = "200" ]; then
  EXECUTION_METRICS_FULL_URL=http://localhost:${EXECUTION_METRICS_PORT}/metrics
fi

export $(cut -d= -f1 /etc/default/execution-variables.env)
export $(cut -d= -f1 /etc/default/beacon-variables.env)
export EXECUTION_METRICS_FULL_URL

envsubst '${NGINX_PROXY_EXECUTION_METRICS_PORT},${EXECUTION_METRICS_FULL_URL}' < /etc/nginx/sites-available/default.template > /etc/nginx/sites-available/default

echo "Configuration for NGINX has been updated."
```
{% endcode %}

```bash
sudo chmod +x ~/nginx_config_script.sh
```

* Edit the service file to run the `~/nginx_config_script.sh` script before every start.

```bash
sudo vim /lib/systemd/system/nginx.service
```

* Edit `USER`

```ini
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/bin/sudo /home/<USER>/nginx_config_script.sh
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

```bash
sudo vim /etc/nginx/sites-available/default.template
```

```nginx
server {
    listen ${NGINX_PROXY_EXECUTION_METRICS_PORT};
    server_name localhost;

    location / {
        proxy_pass ${EXECUTION_METRICS_FULL_URL};
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_cache_bypass $http_upgrade;
        proxy_hide_header Access-Control-Allow-Origin;
    }
}
```

```bash
sudo nginx -t
sudo systemctl restart nginx
```

### Prometheus.yml Template

```bash
vim ~/alerting/prometheus/prometheus.yml.template
```

```yaml
global:
  scrape_interval: 30s
  evaluation_interval: 30s

rule_files:
  - "/etc/prometheus/alert.rules.yml"

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 'localhost:9093'

scrape_configs:
  - job_name: "execution"
    metrics_path: /
    static_configs:
      - targets: ["localhost:${NGINX_PROXY_EXECUTION_METRICS_PORT}"]
  - job_name: "beacon"
    metrics_path: /
    static_configs:
      - targets: ["localhost:${NGINX_PROXY_BEACON_METRICS_PORT}"]
```

## Alerts Config

```bash
vim ~/alerting/prometheus/alert.rules.yml
```

{% code fullWidth="false" %}
```yaml
groups:
- name: ServiceDownAlerts
  rules:
  - alert: ServiceDown
    expr: up == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Service {{ $labels.job }} down"
      description: "{{ $labels.job }} has been down for more than 5 minutes."
```
{% endcode %}

## Alertmanager Config

```bash
vim ~/alerting/alertmanager/alertmanager.yml
```

* Edit `PAGERDUTY_SERVICE_API_KEY`

```yaml
receivers:
- name: 'pagerduty'
  pagerduty_configs:
  - service_key: '<PAGERDUTY_SERVICE_API_KEY>'
    severity: 'critical'
    send_resolved: true

route:
  group_by: ['alertname', 'cluster']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'pagerduty'
  routes:
  - match:
      severity: critical
    receiver: 'pagerduty'
```
