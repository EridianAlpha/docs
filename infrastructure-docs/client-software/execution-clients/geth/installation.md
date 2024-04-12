---
description: Geth client installation guide.
---

# 💾 Installation

### Create Aliases

These aliases make interacting with `Geth` on the command line easier.

{% code fullWidth="true" %}
```bash
echo "alias geth-log='journalctl -f -u geth.service -o cat | ccze -A'" >> ~/.bashrc
echo "alias geth-start='sudo systemctl start geth.service'" >> ~/.bashrc
echo "alias geth-stop='sudo systemctl stop geth.service'" >> ~/.bashrc
echo "alias geth-restart='sudo systemctl restart geth.service'" >> ~/.bashrc
echo "alias geth-status='sudo systemctl status geth.service'" >> ~/.bashrc
echo "alias geth-version='sudo /usr/local/bin/geth --version'" >> ~/.bashrc
echo "alias geth-config='sudo vim /etc/systemd/system/geth.service'" >> ~/.bashrc
echo "alias geth-enable='sudo systemctl enable geth.service'" >> ~/.bashrc
echo "alias geth-disable='sudo systemctl disable geth.service'" >> ~/.bashrc
echo "alias geth-delete-data='sudo rm -rf /var/lib/goethereum/geth'" >> ~/.bashrc
echo "alias geth-update='~/geth-update.sh'" >> ~/.bashrc

echo "alias geth-attach='sudo geth attach --preload ~/geth-console-script.js /var/lib/goethereum/geth.ipc'" >> ~/.bashrc
echo "alias geth-blockNumber='sudo geth --exec \"eth.blockNumber\" attach /var/lib/goethereum/geth.ipc'" >> ~/.bashrc
echo "alias geth-peerCount='sudo geth --exec \"net.peerCount\" attach /var/lib/goethereum/geth.ipc'" >> ~/.bashrc
echo "alias geth-nodeInfo='sudo geth --exec \"admin.nodeInfo\" attach /var/lib/goethereum/geth.ipc'" >> ~/.bashrc

source ~/.bashrc
```
{% endcode %}

### Firewall Configuration

Configure the firewall using generic Execution client UFW settings:[#ufw](../#ufw "mention")

### Go - Install

Find the latest version of `Go` here: [https://go.dev/doc/install](https://go.dev/doc/install)

```bash
GO_LATEST_VERSION=    # Add the latest Go version here

cd ~/
wget https://go.dev/dl/go${GO_LATEST_VERSION}.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go${GO_LATEST_VERSION}.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
echo 'PATH="$PATH:/usr/local/go/bin"' >> ~/.profile
```

### Geth - Install

Build the latest version of `Geth`.

```bash
GETH_VERSION_COMMIT_HASH=        # e.g.3f907d6

cd ~
git clone https://github.com/ethereum/go-ethereum.git
cd go-ethereum
git checkout ${GETH_VERSION_COMMIT_HASH}
make geth
```

Move the compiled `Geth` build to a new directory.

```bash
sudo cp ~/go-ethereum/build/bin/geth /usr/local/bin
```

Create `Geth` user and directory.

```bash
sudo useradd --no-create-home --shell /bin/false goeth
sudo mkdir -p /var/lib/goethereum
```

JWT Secret is now shared between all clients on the same machine:[#create-jwt-secret](../#create-jwt-secret "mention")

### Geth - Configure Service

Set permissions.

```bash
sudo chown -R goeth:goeth /var/lib/goethereum
```

Configure [#execution-service-environment-variables](../#execution-service-environment-variables "mention").

Configure `Geth` service using the command line flags.

```bash
sudo vim /etc/systemd/system/geth.service
```

{% tabs %}
{% tab title="/etc/systemd/system/geth.service" %}
{% code title="/etc/systemd/system/geth.service" %}
```bash
[Unit]
Description=Go Ethereum Client (Geth) - Execution Node
After=network.target
Wants=network.target

[Service]
User=goeth
Group=goeth
Type=simple
Restart=always
RestartSec=5
TimeoutStopSec=1200

EnvironmentFile=/etc/default/execution-variables.env

ExecStart=/usr/local/bin/geth \
    --${NETWORK} \
    --syncmode=snap \
    --port ${P2P_PORT} \
    --discovery.port ${P2P_PORT} \
    --datadir /var/lib/goethereum \
    \
    --pprof \
    --metrics \
    --metrics.expensive \
    --metrics.addr ${METRICS_ADDR} \
    --metrics.port ${METRICS_PORT} \
    \
    --authrpc.jwtsecret=/var/lib/jwtsecret \
    --maxpeers ${MAX_PEERS} \
    \
    --ws \
    --ws.origins '*' \
    --ws.port ${WS_PORT} \
    --ws.addr ${WS_ADDR} \
    \
    --http \
    --http.api "db,eth,net,engine,rpc,web3" \
    --http.vhosts "*" \
    --http.corsdomain "*" \
    --http.addr ${RPC_ADDR} \
    --http.port ${RPC_PORT}

[Install]
WantedBy=default.target
```
{% endcode %}
{% endtab %}

{% tab title="Geth Flags Explained" %}
<table data-header-hidden><thead><tr><th width="294">Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>/usr/local/bin/geth</code></td><td>Starts Geth.</td></tr><tr><td><code>--mainnet</code></td><td>Specifies mainnet as the target network.</td></tr><tr><td><code>--syncmode</code></td><td><ul><li><code>full</code> very/impossibly slow on Mainnet due to Shanghai DDOS attacks.</li><li><code>fast</code> used to be the best option, but is now slower than snap.</li><li><code>snap</code> the current fastest way to sync.</li></ul></td></tr><tr><td><code>--port</code></td><td>Network listening port (TCP).</td></tr><tr><td><code>--discovery.port</code></td><td>UDP port for P2P discovery.</td></tr><tr><td><code>--http</code></td><td>Enable the HTTP-RPC server.</td></tr><tr><td><code>--datadir</code></td><td>Data directory for the databases and keystore.</td></tr><tr><td><code>--metrics</code></td><td>Enable metrics collection and reporting.</td></tr><tr><td><code>--metrics.expensive</code></td><td>Enable expensive metrics collection and reporting.</td></tr><tr><td><code>--pprof</code></td><td><p>Enable the pprof HTTP server.</p><p>Required for metrics to work properly.</p></td></tr><tr><td><code>--http.api</code></td><td>API's offered over the HTTP-RPC interface.</td></tr><tr><td><code>--authrpc.jwtsecret</code></td><td></td></tr><tr><td><code>--maxpeers</code></td><td><p>Maximum number of network peers.</p><ul><li>Network disabled if set to 0.</li><li>Default: 50.</li></ul></td></tr><tr><td><code>--cache</code></td><td><p>Megabytes of memory allocated to internal caching.</p><ul><li>Default = 4096 mainnet full node and 128 light mode.</li></ul></td></tr><tr><td><code>--bootnodes</code></td><td><p>Comma separated enode URLs for P2P discovery bootstrap.</p><ul><li><a href="https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go">https://github.com/ethereum/go-ethereum/blob/master/params/bootnodes.go</a></li></ul></td></tr></tbody></table>
{% endtab %}
{% endtabs %}

Start the service and check it's working as expected.

### Geth - Command Aliases

```bash
daemon-reload   # Reload any changes made to the geth.service
geth-enable     # Enable the geth.service
geth-start      # Start the geth.service
geth-status     # View the status of the geth.service

geth-log        # View the geth.service logs
```

### Geth - Update Scripts

Create `Geth` update script.

```bash
vim ~/geth-update.sh
```

{% code title="~/geth-update.sh" %}
```bash
#!/bin/bash
set -e

while true; do
    read -p "Are you sure you want to update Geth? (Y/N) " yn
    case $yn in
        [Yy]* ) break;;
        [Nn]* ) exit;;
        * ) echo "Please answer Y or N.";;
    esac
done

cd ~/go-ethereum

read -p "Enter the commit hash you want to checkout: " commit_hash

git fetch
git checkout $commit_hash

echo
echo "**************"
echo "Making Geth..."
echo "**************"
make geth

# Check if geth.service is running
service_was_running=0
if sudo systemctl is-active --quiet geth.service; then
    service_was_running=1
    echo "****************"
    echo "Stopping Geth..."
    sudo systemctl stop geth.service
fi

echo "Replacing previous version..."
sudo rm /usr/local/bin/geth
sudo cp ~/go-ethereum/build/bin/geth /usr/local/bin

# Only start geth.service if it was running originally
if [ $service_was_running -eq 1 ]; then
    echo "Restarting Geth..."
    echo "******************"
    sudo systemctl start geth.service
fi
```
{% endcode %}

Make the script executable.

```bash
chmod u+x ~/geth-update.sh
```

### Geth - Configure JavaScript Console

Use `--preload` to load pre-written commands and functions stored in a script file.

```bash
vim ~/geth-console-script.js
```

{% code title="~/geth-console-script.js" %}
```bash
function blockInfo() {
    var blockInfo;
    web3.eth.getBlock(eth.blockNumber, function(e, r) { blockInfo = r; });
    return blockInfo;
}
```
{% endcode %}

Check `Geth` details by attaching to the JavaScript console

```bash
geth-attach
```

`Geth` JavaScript console commands.

```bash
eth.syncing
net.peerCount
```
