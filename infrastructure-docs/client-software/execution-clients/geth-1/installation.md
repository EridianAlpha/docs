---
description: Erigon client installation guide.
---

# 💾 Installation

### Create Aliases

These aliases make interacting with `Erigon` on the command line easier.

{% code fullWidth="true" %}
```bash
echo "alias erigon-log='journalctl -f -u erigon.service -o cat | ccze -A'" >> ~/.bashrc
echo "alias erigon-start='sudo systemctl start erigon.service'" >> ~/.bashrc
echo "alias erigon-stop='sudo systemctl stop erigon.service'" >> ~/.bashrc
echo "alias erigon-restart='sudo systemctl restart erigon.service'" >> ~/.bashrc
echo "alias erigon-status='sudo systemctl status erigon.service'" >> ~/.bashrc

echo "alias erigon-version='sudo /usr/local/bin/erigon --version'" >> ~/.bashrc
echo "alias erigon-config='sudo vim /etc/systemd/system/erigon.service'" >> ~/.bashrc

echo "alias erigon-enable='sudo systemctl enable erigon.service'" >> ~/.bashrc
echo "alias erigon-disable='sudo systemctl disable erigon.service'" >> ~/.bashrc

echo "alias erigon-delete-data='sudo rm -rf /var/lib/goethereum/erigon'" >> ~/.bashrc
echo "alias erigon-update='~/erigon-update.sh'" >> ~/.bashrc

source ~/.bashrc
```
{% endcode %}

### Firewall Configuration

Configure the firewall using generic Execution client UFW settings:[#ufw](../#ufw "mention")

### Erigon - Install

Build the latest version of `Erigon`.

```bash
ERIGON_VERSION_COMMIT_HASH=        # e.g.3f907d6

cd ~
git clone --recurse-submodules https://github.com/ledgerwatch/erigon.git
cd erigon
git checkout ${ERIGON_VERSION_COMMIT_HASH}
make erigon
```

Move the compiled `Erigon` build to a new directory.

```bash
sudo cp ~/erigon/build/bin/erigon /usr/local/bin
```

Create `Erigon` user and directory.

```bash
sudo useradd --no-create-home --shell /bin/false erigon
sudo mkdir -p /var/lib/erigon
```

JWT Secret is now shared between all clients on the same machine:[#create-jwt-secret](../#create-jwt-secret "mention")

### Erigon - Configure Service

Set permissions.

```bash
sudo chown -R erigon:erigon /var/lib/erigon
```

Configure [#execution-service-environment-variables](../#execution-service-environment-variables "mention").

Configure `Erigon` service using the command line flags.

```bash
sudo vim /etc/systemd/system/erigon.service
```

{% tabs %}
{% tab title="/etc/systemd/system/erigon.service" %}
{% code title="/etc/systemd/system/erigon.service" %}
```bash
[Unit]
Description=Erigon Ethereum Client - Execution Node
After=network.target
Wants=network.target

[Service]
User=erigon
Group=erigon
Type=simple
Restart=always
RestartSec=5
TimeoutStopSec=1200

EnvironmentFile=/etc/default/execution-variables.env

ExecStart=/usr/local/bin/erigon \
    --internalcl \
    --chain ${NETWORK} \
    --port ${EXECUTION_P2P_PORT} \
    --datadir /var/lib/erigon \
    \
    --pprof \
    --metrics \
    --metrics.addr ${EXECUTION_METRICS_ADDR} \
    --metrics.port ${EXECUTION_METRICS_PORT} \
    \
    --authrpc.jwtsecret=/var/lib/jwtsecret \
    --maxpeers ${EXECUTION_MAX_PEERS} \
    \
    --ws \
    --ws.port ${EXECUTION_WS_PORT} \
    \
    --http \
    --http.api "eth,erigon,engine" \
    --http.vhosts "*" \
    --http.corsdomain "*" \
    --http.addr ${EXECUTION_RPC_ADDR} \
    --http.port ${EXECUTION_RPC_PORT} \
    \
    --torrent.download.rate=512mb

[Install]
WantedBy=default.target
```
{% endcode %}
{% endtab %}
{% endtabs %}

Start the service and check it's working as expected.

### Erigon - Command Aliases

```bash
daemon-reload     # Reload any changes made to the erigon.service
erigon-enable     # Enable the erigon.service
erigon-start      # Start the erigon.service
erigon-status     # View the status of the erigon.service

erigon-log        # View the erigon.service logs
```

### Erigon - Update Scripts

Create `Erigon` update script.

```bash
vim ~/erigon-update.sh
```

{% code title="~/erigon-update.sh" %}
```bash
#!/bin/bash
set -e

while true; do
    read -p "Are you sure you want to update Erigon? (Y/N) " yn
    case $yn in
        [Yy]* ) break;;
        [Nn]* ) exit;;
        * ) echo "Please answer Y or N.";;
    esac
done

cd ~/erigon

read -p "Enter the commit hash you want to checkout: " commit_hash

git fetch
git checkout $commit_hash

echo
echo "****************"
echo "Making Erigon..."
echo "****************"
make erigon

# Check if erigon.service is running
service_was_running=0
if sudo systemctl is-active --quiet erigon.service; then
    service_was_running=1
    echo "******************"
    echo "Stopping Erigon..."
    sudo systemctl stop erigon.service
fi

echo "Replacing previous version..."
sudo rm /usr/local/bin/erigon
sudo cp ~/erigon/build/bin/erigon /usr/local/bin

# Only start erigon.service if it was running originally
if [ $service_was_running -eq 1 ]; then
    echo "Restarting Erigon..."
    echo "********************"
    sudo systemctl start erigon.service
fi
```
{% endcode %}

Make the script executable.

```bash
chmod u+x ~/erigon-update.sh
```
