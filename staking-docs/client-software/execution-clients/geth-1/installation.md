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
echo "alias besu-version='sudo /usr/local/bin/besu --version'" >> ~/.bashrc
echo "alias besu-config='sudo vim /etc/systemd/system/besu.service'" >> ~/.bashrc
echo "alias besu-enable='sudo systemctl enable besu.service'" >> ~/.bashrc
echo "alias besu-disable='sudo systemctl disable besu.service'" >> ~/.bashrc

echo "alias besu-delete-data='sudo rm -rf /var/lib/??'" >> ~/.bashrc
echo "alias besu-update='~/besu-update.sh'" >> ~/.bashrc

source ~/.bashrc
```
{% endcode %}

### Firewall Configuration

Configure the firewall using generic Execution client UFW settings:[#ufw](../#ufw "mention")

### Dependency - Install

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
tmux new -s besu-build
./gradlew build
```

Wait for the build to finish (\~20 minutes).

Create the binary and exit the `tmux` session.

```bash
./gradlew installDist
exit
```

Move the compiled `Besu` build to a new directory.

```bash
sudo cp ~/besu/build/install/besu/bin/besu /usr/local/bin
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

{% tabs %}
{% tab title="/etc/systemd/system/besu.service" %}
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

ExecStart=/usr/local/bin/besu \
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
    --metrics-port=${METRICS_PORT}
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
{% endtab %}

{% tab title="Besu Flags Explained" %}
<table data-header-hidden><thead><tr><th width="294">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>/usr/local/bin/geth</code></td><td>Starts Geth.</td></tr><tr><td><code>--mainnet</code></td><td>Specifies mainnet as the target network.</td></tr><tr><td><code>--syncmode</code></td><td><ul><li><code>full</code> very/impossibly slow on Mainnet due to Shanghai DDOS attacks.</li><li><code>fast</code> used to be the best option, but is now slower than snap.</li><li><code>snap</code> the current fastest way to sync.</li></ul></td></tr><tr><td><code>--port</code></td><td>Network listening port (TCP).</td></tr><tr><td><code>--discovery.port</code></td><td>UDP port for P2P discovery.</td></tr><tr><td><code>--http</code></td><td>Enable the HTTP-RPC server.</td></tr><tr><td><code>--datadir</code></td><td>Data directory for the databases and keystore.</td></tr><tr><td><code>--metrics</code></td><td>Enable metrics collection and reporting.</td></tr><tr><td><code>--metrics.expensive</code></td><td>Enable expensive metrics collection and reporting.</td></tr><tr><td><code>--pprof</code></td><td><p>Enable the pprof HTTP server.</p><p>Required for metrics to work properly.</p></td></tr><tr><td><code>--http.api</code></td><td>API's offered over the HTTP-RPC interface.</td></tr><tr><td><code>--authrpc.jwtsecret</code></td><td><p>The JWT secret is automatically generated when <code>Geth</code> is first run and added to <code>/var/lib/goethereum/</code></p><p></p><p>This is same location that is pointed to in both the <code>Geth</code> and <code>Lighthouse</code> configuration so that they use the same JWT secret and can talk to each other.</p></td></tr><tr><td><code>--maxpeers</code></td><td><p>Maximum number of network peers.</p><ul><li>Network disabled if set to 0.</li><li>Default: 50.</li></ul></td></tr><tr><td><code>--cache</code></td><td><p>Megabytes of memory allocated to internal caching.</p><ul><li>Default = 4096 mainnet full node and 128 light mode.</li></ul></td></tr><tr><td><code>--bootnodes</code></td><td><p>Comma separated enode URLs for P2P discovery bootstrap.</p><ul><li><a href="https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go">https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go</a></li></ul></td></tr></tbody></table>
{% endtab %}
{% endtabs %}

Start the service and check it's working as expected.

{% tabs %}
{% tab title="Command Aliases" %}
```bash
daemon-reload   # Reload any changes made to the besu.service
besu-enable     # Enable the besu.service
besu-start      # Start the besu.service
besu-status     # View the status of the besu.service

besu-log        # View the besu.service logs
```
{% endtab %}
{% endtabs %}

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
./gradlew build
./gradlew installDist

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
sudo cp ~/besu/build/install/besu/bin/besu /usr/local/bin

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
