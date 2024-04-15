---
description: Teku client installation guide.
---

# 💾 Installation

## Create Aliases

{% code fullWidth="true" %}
```bash
echo "alias teku-version-current='/usr/local/bin/teku/bin/teku --version'" >> ~/.bashrc
echo "alias teku-build='~/teku-build.sh'" >> ~/.bashrc
echo "alias teku-version-new='~/teku/build/install/teku/bin/teku --version'" >> ~/.bashrc
echo "alias teku-deploy='~/teku-deploy.sh'" >> ~/.bashrc

echo "alias teku-beacon-log='journalctl -f -u tekubeacon.service -o cat | ccze -A'" >> ~/.bashrc
echo "alias teku-beacon-start='sudo systemctl start tekubeacon.service'" >> ~/.bashrc
echo "alias teku-beacon-stop='sudo systemctl stop tekubeacon.service'" >> ~/.bashrc
echo "alias teku-beacon-restart='sudo systemctl restart tekubeacon.service'" >> ~/.bashrc
echo "alias teku-beacon-status='sudo systemctl status tekubeacon.service'" >> ~/.bashrc
echo "alias teku-beacon-config='sudo vim /etc/systemd/system/tekubeacon.service'" >> ~/.bashrc
echo "alias teku-beacon-enable='sudo systemctl enable tekubeacon.service'" >> ~/.bashrc
echo "alias teku-beacon-disable='sudo systemctl disable tekubeacon.service'" >> ~/.bashrc
echo "alias teku-beacon-delete-data='sudo rm -rf /var/lib/teku/beacon; sudo mkdir -p /var/lib/teku/beacon; sudo chown -R tekubeacon:tekubeacon /var/lib/teku/beacon'" >> ~/.bashrc

source ~/.bashrc
```
{% endcode %}

## Dependency - Install Java

`Teku` requires version 17+ of Java.

```bash
sudo apt-get install openjdk-17-jre-headless -y
sudo apt-get install openjdk-17-jdk -y

sudo apt-get install libsodium23 libnss3 -y
```

## Teku - Install

Build the latest version of `Teku`.

```bash
TEKU_VERSION_COMMIT_HASH=        # e.g.508459f 

cd ~
git clone https://github.com/Consensys/teku.git
cd teku
git checkout ${TEKU_VERSION_COMMIT_HASH}
./gradlew installDist
```

Move the compiled `Teku` build to a new directory.

```bash
sudo cp -R ~/teku/build/install/teku /usr/local/bin
```

Check version.

```bash
teku-version-current
```

Create `Teku` directory.

```bash
sudo mkdir -p /var/lib/teku
```

## Firewall Configuration

Configure the firewall using generic Beacon client UFW settings: [#ufw-config](../#ufw-config "mention")

## Teku BN - Configure Service

Create `tekubeacon` user and set permissions.

```bash
sudo useradd --no-create-home --shell /bin/false tekubeacon
sudo mkdir -p /var/lib/teku/
sudo chown -R tekubeacon:tekubeacon /var/lib/teku/
```

Configure [#beacon-service-environment-variables](../#beacon-service-environment-variables "mention").

Configure `tekubeacon` service using the command line flags.

```bash
sudo vim /etc/systemd/system/tekubeacon.service
```

{% code title="tekubeacon.service" %}
```bash
[Unit]
Description=Teku Ethereum Client - Beacon Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=tekubeacon
Group=tekubeacon
Restart=always
RestartSec=5

EnvironmentFile=/etc/default/beacon-variables.env

ExecStart=/usr/local/bin/teku/bin/teku \
    --network ${NETWORK} \
    --data-path /var/lib/teku/ \
    --ee-jwt-secret-file="/var/lib/jwtsecret" \
    --ee-endpoint ${BEACON_EXECUTION_ENDPOINTS} \
    --p2p-port ${BEACON_P2P_PORT} \
    --p2p-peer-lower-bound=150 \
    --p2p-peer-upper-bound=175 \
    \
    --rest-api-enabled=true \
    --rest-api-interface=${BEACON_HTTP_ADDRESS} \
    --rest-api-port=${BEACON_HTTP_PORT} \
    --rest-api-host-allowlist "*" \
    \
    --metrics-enabled=true \
    --metrics-interface ${BEACON_METRICS_ADDR} \
    --metrics-port ${BEACON_METRICS_PORT} \
    --metrics-host-allowlist "*" \
    \
    --validators-proposer-default-fee-recipient ${BEACON_SUGGESTED_FEE_RECIPIENT} \
    --initial-state ${BEACON_CHECKPOINT_SYNC_URL} \
    --builder-endpoint ${BEACON_BUILDER} \
    --validators-builder-registration-default-enabled=true

[Install]
WantedBy=multi-user.target
```
{% endcode %}

Start the service and check it's working as expected.

## Command Aliases

```bash
daemon-reload          # Reload any changes made to the tekubeacon.service
teku-beacon-enable     # Enable the tekubeacon.service
teku-beacon-start      # Start the tekubeacon.service
teku-beacon-status     # View the status of the tekubeacon.service

teku-beacon-log        # View the tekubeacon.service logs
```

## Teku - Update Scripts

Create `Teku` build script.

```bash
vim ~/teku-build.sh
```

{% code title="~/teku-build.sh" %}
```bash
#!/bin/bash
set -e
cd ~/teku

read -p "Enter the commit hash you want to checkout: " commit_hash

git fetch
git checkout $commit_hash

echo
echo "****************"
echo "Building Teku..."
echo "****************"
./gradlew installDist
```
{% endcode %}

Create `Teku` deploy script.

```bash
vim ~/teku-deploy.sh
```

{% code title="~/teku-deploy.sh" %}
```bash
#!/bin/bash
# set -e     # This isn't working correctly, so comment out for now

while true; do
    read -p "Are you sure you want to deploy Teku? (Y/N) " yn
    case $yn in
        [Yy]* ) break;;
        [Nn]* ) exit;;
        * ) echo "Please answer Y or N.";;
    esac
done

# Check if the services exist before checking their status
if systemctl list-units --full -all | grep -Fq tekubeacon.service; then
    beacon_status=$(sudo systemctl is-active tekubeacon.service)
else
    beacon_status="not_installed"
fi

if systemctl list-units --full -all | grep -Fq tekuvalidator.service; then
    validator_status=$(sudo systemctl is-active tekuvalidator.service)
else
    validator_status="not_installed"
fi

echo "**********************"

if [ "$beacon_status" = "active" ]; then
    echo "Stopping Teku Beacon..."
    sudo systemctl stop tekubeacon.service
elif [ "$beacon_status" = "not_installed" ]; then
    echo "Warning: Teku Beacon service is not installed."
fi

if [ "$validator_status" = "active" ]; then
    echo "Stopping Teku Validator..."
    sudo systemctl stop tekuvalidator.service
elif [ "$validator_status" = "not_installed" ]; then
    echo "Warning: Teku Validator service is not installed."
fi

echo "Replacing previous version..."

# Check if the directory exists before trying to remove it
if [ -d /usr/local/bin/teku ]; then
    sudo rm -rf /usr/local/bin/teku
fi

# Check if the source directory exists before copying
if [ -d ~/teku/build/install/teku ]; then
    sudo cp -R ~/teku/build/install/teku /usr/local/bin
else
    echo "Error: Source directory does not exist. Installation aborted."
    exit 1
fi

if [ "$beacon_status" = "active" ]; then
    echo "Restarting Teku Beacon..."
    sudo systemctl start tekubeacon.service
fi

if [ "$validator_status" = "active" ]; then
    echo "Restarting Teku Validator..."
    sudo systemctl start tekuvalidator.service
fi
```
{% endcode %}

Make all scripts executable.

```bash
chmod u+x ~/teku-build.sh
chmod u+x ~/teku-deploy.sh
```
