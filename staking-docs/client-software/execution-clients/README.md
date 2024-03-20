# ⚙️ Execution Clients

## Process - Changing Clients

1. Stop old client.
2. Disable old client.
3. Delete old client data.
4. Enable new client.
5. Start new client.

## UFW

Configure the firewall.

{% code title="Execution clients" %}
```bash
EXECUTION_P2P_PORT=        # Geth default: 30303
EXECUTION_WS_PORT=         # Geth default: 8546
EXECUTION_METRICS_PORT=    # Geth default: 6060
EXECUTION_RPC_PORT=        # Geth default: 8545

sudo ufw allow ${EXECUTION_P2P_PORT} comment 'Allow Execution P2P in'
sudo ufw allow ${EXECUTION_WS_PORT} comment 'Allow Execution WS in'
sudo ufw allow ${EXECUTION_METRICS_PORT} comment 'Allow Execution Metrics in'
sudo ufw allow ${EXECUTION_RPC_PORT} comment 'Allow Execution RPC Port in'
```
{% endcode %}

## Create JWT Secret.

This is now shared between all clients on the same machine.

```bash
sudo openssl rand -hex 32 | tr -d "\n" > "/tmp/jwtsecret"
sudo mv /tmp/jwtsecret /var/lib/
sudo chmod +r /var/lib/jwtsecret
```

## Execution Service Environment Variables

```bash
sudo vim /etc/default/execution-variables.env
```

```ini
NETWORK=         # E.g. mainnet or holesky
P2P_PORT=        # Default: 30303
MAX_PEERS=       # Default: 50
WS_ADDR=         # e.g. 0.0.0.0
WS_PORT=         # Default: 8546
RPC_ADDR=        # e.g. 0.0.0.0
RPC_PORT=        # Default: 8545
METRICS_ADDR=    # e.g. 0.0.0.0
METRICS_PORT=    # Default: 6060
```
