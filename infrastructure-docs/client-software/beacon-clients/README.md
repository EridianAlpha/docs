# 💎 Beacon Clients

### UFW

Configure the firewall.

{% code title="Beacon clients" %}
```bash
LIGHTHOUSE_P2P_PORT=              # Default: 9000
LIGHTHOUSE_HTTP_PORT=             # Default: 5052
LIGHTHOUSE_METRICS_PORT=          # Default: 5054

sudo ufw allow ${LIGHTHOUSE_P2P_PORT} comment 'Allow Lighthouse P2P in'
sudo ufw allow ${LIGHTHOUSE_HTTP_PORT} comment 'Allow Lighthouse http in'
sudo ufw allow ${LIGHTHOUSE_METRICS_PORT} comment 'Allow Lighthouse Metrics in'
```
{% endcode %}
