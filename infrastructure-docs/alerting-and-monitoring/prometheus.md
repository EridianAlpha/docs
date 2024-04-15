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

```bash
vim ~/alerting/prometheus/prometheus.yml.template
```

```yaml
global:
  scrape_interval: 30s
  evaluation_interval: 30s

rule_files:
  - "alert.rules.yml"

scrape_configs:
  - job_name: "execution"
    metrics_path: /debug/metrics/prometheus
    static_configs:
      - targets: ["localhost:${EXECUTION_METRICS_PORT}"]
  - job_name: "beacon"
    metrics_path: /metrics
    static_configs:
      - targets: ["localhost:${BEACON_METRICS_PORT}"]
```

## Alerts Config

```bash
vim ~/alerting/prometheus/alert.rules.yml
```

{% code fullWidth="true" %}
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
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
```
{% endcode %}

## Alertmanager Config

```bash
vim ~/alerting/alertmanager/alertmanager.yml
```

```yaml
receivers:
- name: 'pagerduty'
  pagerduty_configs:
  - service_key: '<PAGEDUTY_SERVICE_API_KEY>'
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

