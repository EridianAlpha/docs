---
description: Besu client installation guide.
---

# 💾 Installation

### Create Aliases

These aliases make interacting with `Besu` on the command line easier.

{% code fullWidth="true" %}
```bash
echo "alias besu-log='journalctl -f -u besu.service -o cat | ccze -A'" >> ~/.bashrc
echo "alias besu-start='sudo systemctl start besu.service'" >> ~/.bashrc
echo "alias besu-stop='sudo systemctl stop besu.service'" >> ~/.bashrc
echo "alias besu-restart='sudo systemctl restart besu.service'" >> ~/.bashrc
echo "alias besu-status='sudo systemctl status besu.service'" >> ~/.bashrc
echo "alias besu-version='sudo /usr/local/bin/besu/bin/besu --version'" >> ~/.bashrc
echo "alias besu-config='sudo vim /etc/systemd/system/besu.service'" >> ~/.bashrc
echo "alias besu-enable='sudo systemctl enable besu.service'" >> ~/.bashrc
echo "alias besu-disable='sudo systemctl disable besu.service'" >> ~/.bashrc
echo "alias besu-delete-data='sudo rm -rf /var/lib/besu; sudo mkdir -p /var/lib/besu; sudo chown -R besu:besu /var/lib/besu'" >> ~/.bashrc
echo "alias besu-update='~/besu-update.sh'" >> ~/.bashrc

source ~/.bashrc
```
{% endcode %}

### Firewall Configuration

Configure the firewall using generic Execution client UFW settings:[#ufw](../#ufw "mention")

### Dependency - Install Java

`Besu` requires version 17+ of Java: [https://besu.hyperledger.org/public-networks/get-started/install/binary-distribution#prerequisites-1](https://besu.hyperledger.org/public-networks/get-started/install/binary-distribution#prerequisites-1)

```bash
sudo apt-get install openjdk-17-jre-headless -y
sudo apt-get install openjdk-17-jdk -y

sudo apt-get install libsodium23 libnss3 -y
```

### Besu - Install

Build the latest version of `Besu`.

```bash
BESU_VERSION_COMMIT_HASH=        # e.g.3f907d6

cd ~
git clone --recursive https://github.com/hyperledger/besu
cd ~/besu
git checkout ${BESU_VERSION_COMMIT_HASH}
./gradlew build -x test
./gradlew clean installDist
```

Move the compiled `Besu` build to a new directory.

```bash
sudo cp -R ~/besu/build/install/besu /usr/local/bin
```

Check version.

```bash
/usr/local/bin/besu/bin/besu --version
```

Create `Besu` user and directory.

```bash
sudo useradd --no-create-home --shell /bin/false besu
sudo mkdir -p /var/lib/besu
```

JWT Secret is now shared between all clients on the same machine:[#create-jwt-secret](../#create-jwt-secret "mention")

### Besu - Configure Service

Set permissions.

```bash
sudo chown -R besu:besu /var/lib/besu
```

Configure [#execution-service-environment-variables](../#execution-service-environment-variables "mention").

Configure `Besu` service using the command line flags.

```bash
sudo vim /etc/systemd/system/besu.service
```

{% code title="/etc/systemd/system/besu.service" %}
```bash
[Unit]
Description=Besu Ethereum Client - Execution Node
After=network.target
Wants=network.target

[Service]
User=besu
Group=besu
Type=simple
Restart=always
RestartSec=5
TimeoutStopSec=1200

EnvironmentFile=/etc/default/execution-variables.env

ExecStart=/usr/local/bin/besu/bin/besu \
    --network=${NETWORK} \
    --sync-mode=SNAP \
    --engine-jwt-secret=/var/lib/jwtsecret \
    --data-path=/var/lib/besu \
    --data-storage-format=BONSAI \
    --p2p-port=${P2P_PORT} \
    --max-peers=${MAX_PEERS} \
    --engine-host-allowlist="*" \
    --host-allowlist="*" \
    \
    --metrics-enabled=true \
    --metrics-host=${METRICS_ADDR} \
    --metrics-port=${METRICS_PORT} \
    \
    --rpc-ws-enabled=true \
    --rpc-ws-api=ETH,NET,WEB3 \
    --rpc-ws-host=${WS_ADDR} \
    --rpc-ws-port=${WS_PORT} \
    \
    --rpc-http-enabled=true \
    --rpc-http-api=ETH,NET,WEB3 \
    --rpc-http-cors-origins="*" \
    --rpc-http-host=${RPC_ADDR} \
    --rpc-http-port=${RPC_PORT} \
    \
    --Xplugin-rocksdb-high-spec-enabled

[Install]
WantedBy=default.target
```
{% endcode %}

Start the service and check it's working as expected.

### Besu - Command Aliases

```bash
daemon-reload   # Reload any changes made to the besu.service
besu-enable     # Enable the besu.service
besu-start      # Start the besu.service
besu-status     # View the status of the besu.service

besu-log        # View the besu.service logs
```

### Besu - Update Scripts

Create `Besu` update script.

```bash
vim ~/besu-update.sh
```

{% code title="~/besu-update.sh" %}
```bash
#!/bin/bash
set -e

while true; do
    read -p "Are you sure you want to update Besu? (Y/N) " yn
    case $yn in
        [Yy]* ) break;;
        [Nn]* ) exit;;
        * ) echo "Please answer Y or N.";;
    esac
done

cd ~/besu

read -p "Enter the commit hash you want to checkout: " commit_hash

git fetch
git checkout $commit_hash

echo
echo "**************"
echo "Making Besu..."
echo "**************"
./gradlew build -x test
./gradlew clean installDist

# Check if besu.service is running
service_was_running=0
if sudo systemctl is-active --quiet besu.service; then
    service_was_running=1
    echo "****************"
    echo "Stopping Besu..."
    sudo systemctl stop besu.service
fi

echo "Replacing previous version..."
sudo rm /usr/local/bin/besu
sudo cp -R ~/besu/build/install/besu /usr/local/bin

# Only start besu.service if it was running originally
if [ $service_was_running -eq 1 ]; then
    echo "Restarting Besu..."
    echo "******************"
    sudo systemctl start besu.service
fi
```
{% endcode %}

Make the script executable.

```bash
chmod u+x ~/besu-update.sh
```
