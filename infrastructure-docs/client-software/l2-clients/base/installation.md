---
description: Base Client installation guide.
---

# 💾 Installation

{% hint style="info" %}
🏗️ Under Construction 🏗️
{% endhint %}

## Dependency - Install Docker

[#install-docker](../../../linux-software/installation.md#install-docker "mention")

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

* Set
  * `env_file:`
* Modify host port mappings as required

## Snapshot Download

```bash
wget https://mainnet-full-snapshots.base.org/$(curl https://mainnet-full-snapshots.base.org/latest)
```

<mark style="color:yellow;">**TODO: What to do with the download?**</mark>

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

