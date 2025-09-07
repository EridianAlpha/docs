# 🌡️ HealthChecks.io

## Create a new check

{% embed url="https://healthchecks.io" %}

* These settings allow for a system restart without triggering the alerts
* Period:
  * 5 minutes
  * The expected time between pings
* Grace Time:
  * 10 minutes
  * When a check is late, how long to wait to send an alert

## Bash script

Used to monitor if Prometheus and Alertmanager are running, and if the entire machine is offline or unresponsive.

```bash
sudo vim /etc/default/healthchecks.sh
```

* Edit `UNIQUE_ID`

```bash
#!/bin/bash

# Healthchecks.io URL
# ***********
# CHANGE HERE
# ↓↓↓↓↓↓↓↓↓↓↓
healthchecks_io_url="https://hc-ping.com/<UNIQUE_ID>"

# URLs for the health endpoints of Prometheus and Alertmanager
prometheus_health_url="http://localhost:9090/-/healthy"
alertmanager_health_url="http://localhost:9093/-/healthy"

# Check health of Prometheus
if curl -f ${prometheus_health_url}; then
  # Check health of Alertmanager
  if curl -f ${alertmanager_health_url}; then
    # Send heartbeat to Healthchecks.io
    curl -fsS --retry 3 ${healthchecks_io_url} > /dev/null
  else
    echo "Alertmanager is not healthy."
  fi
else
  echo "Prometheus is not healthy."
fi
```

```bash
sudo chmod u+x /etc/default/healthchecks.sh
```

## Configure CRON

* Run every 1 minute.

```bash
sudo crontab -e
```

```
* * * * * /etc/default/healthchecks.sh
```

## Integrations

Integrates with [pagerduty.md](pagerduty.md "mention") for alerts.
