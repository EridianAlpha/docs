# 🤝 Beacon Clients

{% content-ref url="lighthouse/" %}
[lighthouse](lighthouse/)
{% endcontent-ref %}

{% content-ref url="lighthouse-1/" %}
[lighthouse-1](lighthouse-1/)
{% endcontent-ref %}

## Process - Changing Clients

1. Stop old client.
2. Disable old client.
3. Delete old client data.
4. Enable new client.
5. Start new client.

## UFW Config

Configure the firewall.

{% code title="Beacon Clients" %}
```bash
BEACON_P2P_PORT=              # Default: 9000
BEACON_HTTP_PORT=             # Default: 5052
BEACON_METRICS_PORT=          # Default: 5054

sudo ufw allow ${BEACON_P2P_PORT} comment 'Allow Beacon P2P in'
sudo ufw allow ${BEACON_HTTP_PORT} comment 'Allow Beacon http in'
sudo ufw allow ${BEACON_METRICS_PORT} comment 'Allow Beacon Metrics in'
```
{% endcode %}

## Beacon Service Environment Variables

```bash
sudo vim /etc/default/beacon-variables.env
```

```ini
NETWORK=                            # E.g. mainnet or holesky
BEACON_EXECUTION_ENDPOINTS=         # E.g. http://127.0.0.1:8551
BEACON_P2P_PORT=                    # Default: 9000
BEACON_HTTP_ADDRESS=                # E.g. 0.0.0.0
BEACON_HTTP_PORT=                   # Default: 5052
BEACON_METRICS_ADDR=                # E.g. 0.0.0.0
BEACON_METRICS_PORT=                # E.g. 5054
BEACON_SUGGESTED_FEE_RECIPIENT=     # E.g. 0x0000...
BEACON_CHECKPOINT_SYNC_URL=         # E.g. https://beaconstate.ethstaker.cc
BEACON_BUILDER=                     # E.g. http://127.0.0.1:18550
NGINX_PROXY_BEACON_METRICS_PORT=    # Unified metrics port e.g. 5055
```
