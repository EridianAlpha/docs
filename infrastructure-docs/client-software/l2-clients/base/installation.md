---
description: Base Client installation guide.
---

# 💾 Installation

{% hint style="info" %}
🏗️ Under Construction 🏗️
{% endhint %}

## Create Aliases

{% code fullWidth="true" %}
```bash
echo "alias base-log='cd ~/node; docker compose logs -f --tail 100'" >> ~/.bashrc
echo "alias base-start='cd ~/node; docker compose up -d'" >> ~/.bashrc
echo "alias base-stop='cd ~/node; docker compose down'" >> ~/.bashrc
echo "alias base-restart='cd ~/node; docker compose down; docker compose up -d'" >> ~/.bashrc
echo "alias base-config='cd ~/node; vim docker-compose.yml'" >> ~/.bashrc

source ~/.bashrc
```
{% endcode %}

## Dependency - Install Docker

[#install-docker](../../../linux-software/installation.md#install-docker "mention")

## UFW Config

Configure the firewall.

{% code title="Execution Clients" %}
```bash
BASE_GETH_P2P_PORT=        # Default: 30303
BASE_NODE_P2P_PORT=        # Default: 9222

sudo ufw allow ${BASE_GETH_P2P_PORT} comment 'Allow Base Geth P2P in'
sudo ufw allow ${BASE_NODE_P2P_PORT} comment 'Allow Base Node P2P in'
```
{% endcode %}

## Base - Install

{% embed url="https://docs.base.org/tutorials/run-a-base-node" %}

```bash
git clone https://github.com/base-org/node.git
```

* Edit the `.env.mainnet`

```bash
vim .env.mainnet
```

* Set
  * `OP_NODE_L1_ETH_RPC`
  * `OP_NODE_L1_BEACON`
  * `OP_NODE_L1_TRUST_RPC=true`
* Edit the `docker-compose.yml`

```bash
vim docker-compose.yml
```

* Set to mainnet
  * `env_file:`
* Add `restart: unless-stopped` to all services
* Modify host port mappings as required

## Snapshot Download

```bash
mkdir ~/base-snapshot-download
cd ~/base-snapshot-download
tmux new -s base-snapshot-download

wget https://mainnet-full-snapshots.base.org/$(curl https://mainnet-full-snapshots.base.org/latest)
```

```bash
base-stop
sudo rm -rf ~/node/geth-data/geth
sudo mv ~/base-snapshot-download/snapshots/mainnet/download/geth ~/node/geth-data/geth
```

## Docker Commands

```bash
docker compose up -d
docker compose logs -f
docker compose logs -f geth
docker compose logs -f node
```

## Check sync progress

{% code fullWidth="false" %}
```bash
timestamp=$(curl -d '{"id":0,"jsonrpc":"2.0","method":"optimism_syncStatus"}' \
  -H "Content-Type: application/json" http://localhost:7545 | \
  jq -r '.result.unsafe_l2.timestamp')

if [[ -z "$timestamp" ]]; then
  echo "Failed to retrieve timestamp"
else
  echo
  echo
  echo Latest synced block behind by: $((($(date +%s) - timestamp) / 60 / 60 / 24)) days
  echo
  echo
fi
```
{% endcode %}
