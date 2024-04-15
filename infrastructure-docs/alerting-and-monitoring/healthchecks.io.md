# 🌡️ HealthChecks.io

{% embed url="https://healthchecks.io" %}

Used to monitor if Prometheus and Alertmanager are running, and if the entire machine is offline or unresponsive.

```bash
vim ~/healthchecks.sh
```

```bash
#!/bin/bash

# URLs for the health endpoints of Prometheus and Alertmanager
prometheus_health_url="http://localhost:9090/-/healthy"
alertmanager_health_url="http://localhost:9093/-/healthy"

# Healthchecks.io URL
healthchecks_io_url="https://hc-ping.com/your-unique-uuid"

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
chmod u+x ~/healthchecks.sh
```

## Configure CRON

```bash
sudo crontab -e
```

```
*/5 * * * * /home/<USERNAME>/healthchecks.sh
```

## Integrations

Integrates with [pagerduty.md](pagerduty.md "mention") for alerts.
