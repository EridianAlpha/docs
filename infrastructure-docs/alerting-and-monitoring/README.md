# 🚨 Alerting and Monitoring

Each machine runs a Prometheus and Alertmanager instance which monitors the services running on the same machine.

If Prometheus and/or Alertmanager are down, or the entire machine is down/unresponsive then HealthChecks.io is used as a dead-mans-hand alert. Periodic heartbeat pings are sent by a cron script (`~/healthchecks.sh`) every 5 minutes, with a 10 minute grace period.

All services are integrated with PagerDuty for alerts.

{% content-ref url="prometheus.md" %}
[prometheus.md](prometheus.md)
{% endcontent-ref %}

{% content-ref url="healthchecks.io.md" %}
[healthchecks.io.md](healthchecks.io.md)
{% endcontent-ref %}

{% content-ref url="pagerduty.md" %}
[pagerduty.md](pagerduty.md)
{% endcontent-ref %}
