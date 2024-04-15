---
description: Lighthouse client installation guide.
---

# 💾 Installation

## Create Aliases

{% code fullWidth="true" %}
```bash
echo "alias lighthouse-version-current='/usr/local/bin/lighthouse --version'" >> ~/.bashrc
echo "alias lighthouse-build='~/lighthouse-build.sh'" >> ~/.bashrc
echo "alias lighthouse-version-new='~/.cargo/bin/lighthouse --version'" >> ~/.bashrc
echo "alias lighthouse-deploy='~/lighthouse-deploy.sh'" >> ~/.bashrc

echo "alias lighthouse-beacon-log='journalctl -f -u lighthousebeacon.service -o cat | ccze -A'" >> ~/.bashrc
echo "alias lighthouse-beacon-start='sudo systemctl start lighthousebeacon.service'" >> ~/.bashrc
echo "alias lighthouse-beacon-stop='sudo systemctl stop lighthousebeacon.service'" >> ~/.bashrc
echo "alias lighthouse-beacon-restart='sudo systemctl restart lighthousebeacon.service'" >> ~/.bashrc
echo "alias lighthouse-beacon-status='sudo systemctl status lighthousebeacon.service'" >> ~/.bashrc
echo "alias lighthouse-beacon-config='sudo vim /etc/systemd/system/lighthousebeacon.service'" >> ~/.bashrc
echo "alias lighthouse-beacon-enable='sudo systemctl enable lighthousebeacon.service'" >> ~/.bashrc
echo "alias lighthouse-beacon-disable='sudo systemctl disable lighthousebeacon.service'" >> ~/.bashrc
echo "alias lighthouse-beacon-delete-data='sudo rm -rf /var/lib/lighthouse/beacon; sudo mkdir -p /var/lib/lighthouse/beacon; sudo chown -R lighthousebeacon:lighthousebeacon /var/lib/lighthouse/beacon'" >> ~/.bashrc

source ~/.bashrc
```
{% endcode %}

## Dependency - Install Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

<1>
<ENTER>

source $HOME/.cargo/env
```

Make sure `protoc` is installed otherwise the build will fail.

```bash
sudo apt-get install -y protobuf-compiler
protoc --version
```

## Lighthouse - Install

Build the latest version of `Lighthouse`.

```bash
LIGHTHOUSE_VERSION_COMMIT_HASH=        # e.g.441fc16 

cd ~
git clone https://github.com/sigp/lighthouse.git
cd ~/lighthouse
git checkout ${LIGHTHOUSE_VERSION_COMMIT_HASH}
make
```

Move the compiled `Lighthouse` build to a new directory.

```bash
sudo cp ~/.cargo/bin/lighthouse /usr/local/bin
```

Check version.

```bash
/usr/local/bin/lighthouse --version
```

Create `Lighthouse` directory.

```bash
sudo mkdir -p /var/lib/lighthouse
```

## Firewall Configuration

Configure the firewall using generic Beacon client UFW settings: [#ufw-config](../#ufw-config "mention")

## Lighthouse BN - Configure Service

Create `lighthousebeacon` user and set permissions.

```bash
sudo useradd --no-create-home --shell /bin/false lighthousebeacon
sudo mkdir -p /var/lib/lighthouse/beacon
sudo chown -R lighthousebeacon:lighthousebeacon /var/lib/lighthouse/beacon
```

Configure [#beacon-service-environment-variables](../#beacon-service-environment-variables "mention").

Configure `lighthousebeacon` service using the command line flags.

```bash
sudo vim /etc/systemd/system/lighthousebeacon.service
```

{% tabs %}
{% tab title="lighthousebeacon.service" %}
{% code title="lighthousebeacon.service" %}
```bash
[Unit]
Description=Lighthouse Ethereum Client - Beacon Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=lighthousebeacon
Group=lighthousebeacon
Restart=always
RestartSec=5

EnvironmentFile=/etc/default/beacon-variables.env

ExecStart=/usr/local/bin/lighthouse bn \
    --network ${NETWORK} \
    --datadir /var/lib/lighthouse \
    --jwt-secrets="/var/lib/jwtsecret" \
    --execution-endpoints ${BEACON_EXECUTION_ENDPOINTS} \
    --port ${BEACON_P2P_PORT} \
    \
    --http \
    --http-address=${BEACON_HTTP_ADDRESS} \
    --http-port=${BEACON_HTTP_PORT} \
    --http-allow-origin "*" \
    \
    --metrics \
    --metrics-address ${BEACON_METRICS_ADDR} \
    --metrics-port ${BEACON_METRICS_PORT} \
    --metrics-allow-origin "*" \
    --validator-monitor-auto \
    \
    --suggested-fee-recipient ${BEACON_SUGGESTED_FEE_RECIPIENT} \
    --checkpoint-sync-url ${BEACON_CHECKPOINT_SYNC_URL} \
    --builder ${BEACON_BUILDER} \
    --reconstruct-historic-states

[Install]
WantedBy=multi-user.target
```
{% endcode %}
{% endtab %}

{% tab title="Lighthouse BN Flags Explained" %}
| `/usr/local/bin/lighthouse bn`                                                 | Starts the `Beacon` node.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `--network`                                                                    | Name of the chain Lighthouse will sync and follow (e.g. `mainnet` or `goerli`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `--datadir`                                                                    | Used to specify a custom root data directory for lighthouse keys and databases.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `--port`                                                                       | The TCP/UDP port to listen on for peer discovery.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `--http`                                                                       | <p>Enable the RESTful HTTP API server.</p><ul><li>Disabled by default.</li></ul>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `--http-address`                                                               | Specify the listening address of the server.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `--http-port`                                                                  | Specify the listening port of the server.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `--metrics`                                                                    | <p>Enable the Prometheus metrics HTTP server.</p><ul><li>Disabled by default.</li></ul>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `--validator-monitor-auto`                                                     | <p>When the <code>--validator-monitor-auto</code> flag is supplied, any validator which uses the <code>beacon_committee_subscriptions</code> API endpoint will be enrolled for additional monitoring.</p><ul><li>All active validators will use this endpoint each epoch, so you can expect it to detect all local and active validators within several minutes after start up.</li><li><a href="https://lighthouse-book.sigmaprime.io/validator-monitoring.html#automatic">https://lighthouse-book.sigmaprime.io/validator-monitoring.html#automatic</a></li></ul>                        |
| <p><code>--monitoring-endpoint</code><br><code>(Not curreclty used)</code></p> | <p>Enables the monitoring service for sending system metrics to a remote endpoint.</p><ul><li>This can be used to monitor your setup on certain services (e.g. <a href="https://beaconcha.in/user/settings#app">https://beaconcha.in/user/settings#app</a>).</li><li>This flag sets the endpoint where the beacon node metrics will be sent.</li><li>Note: This will send information to a remote server which may identify and associate your validators, IP address, and other personal information.</li><li>Always use a HTTPS connection and never provide an untrusted URL.</li></ul> |
| `--execution-endpoints`                                                        | <p>One or more comma-delimited server endpoints for HTTP JSON-RPC connection.</p><ul><li>If multiple endpoints are given the endpoints are used as fallback in the given order.</li></ul>                                                                                                                                                                                                                                                                                                                                                                                                  |
| `--jwt-secrets`                                                                | <p>The JWT secret is automatically generated when <code>Geth</code> is first run and added to <code>/var/lib/goethereum/</code></p><ul><li>This is same location that is pointed to in both the <code>Geth</code> and <code>Lighthouse</code> configuration so that they use the same JWT secret and can talk to each other.</li></ul>                                                                                                                                                                                                                                                     |
| `--suggested-fee-recipient`                                                    | <p>This address receives transaction fees collected from any blocks produced by this node.</p><ul><li>The <code>--suggested-fee-recipient</code> can be provided to the <code>Beacon Node</code> to act as a default value when the validator client does not transmit a <code>suggested_fee_recipient</code> to the <code>Beacon Node</code></li></ul>                                                                                                                                                                                                                                    |
| `--checkpoint-sync-url`                                                        | <p>Used to fast sync the chain rather than starting at genesis.</p><ul><li><a href="https://lighthouse-book.sigmaprime.io/checkpoint-sync.html#automatic-checkpoint-sync">https://lighthouse-book.sigmaprime.io/checkpoint-sync.html#automatic-checkpoint-sync</a></li><li>Only needed for the initial sync, I can remove it after.</li></ul>                                                                                                                                                                                                                                              |
| `--builder`                                                                    | <p>Used to query the provided URL during block production for a block payload with stubbed-out transactions.</p><ul><li>If this request fails, Lighthouse will fall back to the local execution engine and produce a block using transactions gathered and verified locally.</li></ul>                                                                                                                                                                                                                                                                                                     |
| `--reconstruct-historic-states`                                                | <p>Used to reconstruct the history of the beacon chain when checkpoint sync was used.</p><ul><li>Allows access to all historical data.</li><li><a href="https://lighthouse-book.sigmaprime.io/checkpoint-sync.html#reconstructing-states">https://lighthouse-book.sigmaprime.io/checkpoint-sync.html#reconstructing-states</a></li></ul>                                                                                                                                                                                                                                                   |
| `--purge-db`                                                                   | Add this flag to delete the existing database.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
{% endtab %}
{% endtabs %}

Start the service and check it's working as expected.

## Command Aliases

```bash
daemon-reload     # Reload any changes made to the lighthousebeacon.service
lighthouse-beacon-enable     # Enable the lighthousebeacon.service
lighthouse-beacon-start      # Start the lighthousebeacon.service
lighthouse-beacon-status     # View the status of the lighthousebeacon.service

lighthouse-beacon-log        # View the lighthousebeacon.service logs
```

## Lighthouse - Update Scripts

Create `Lighthouse` build script.

```bash
vim ~/lighthouse-build.sh
```

{% code title="~/lighthouse-build.sh" %}
```bash
#!/bin/bash
set -e
cd ~/lighthouse

read -p "Enter the commit hash you want to checkout: " commit_hash

git fetch
git checkout $commit_hash

echo
echo "********************"
echo "Making Lighthouse..."
echo "********************"
make
```
{% endcode %}

Create `Lighthouse` deploy script.

```bash
vim ~/lighthouse-deploy.sh
```

{% code title="~/lighthouse-deploy.sh" %}
```bash
#!/bin/bash
# set -e     # This isn't working correctly, so comment out for now

while true; do
    read -p "Are you sure you want to deploy Lighthouse? (Y/N) " yn
    case $yn in
        [Yy]* ) break;;
        [Nn]* ) exit;;
        * ) echo "Please answer Y or N.";;
    esac
done

# Check if the services exist before checking their status
if systemctl list-units --full -all | grep -Fq lighthousebeacon.service; then
    beacon_status=$(sudo systemctl is-active lighthousebeacon.service)
else
    beacon_status="not_installed"
fi

if systemctl list-units --full -all | grep -Fq lighthousevalidator.service; then
    validator_status=$(sudo systemctl is-active lighthousevalidator.service)
else
    validator_status="not_installed"
fi

echo "**********************"

if [ "$beacon_status" = "active" ]; then
    echo "Stopping Lighthouse Beacon..."
    sudo systemctl stop lighthousebeacon.service
elif [ "$beacon_status" = "not_installed" ]; then
    echo "Warning: Lighthouse Beacon service is not installed."
fi

if [ "$validator_status" = "active" ]; then
    echo "Stopping Lighthouse Validator..."
    sudo systemctl stop lighthousevalidator.service
elif [ "$validator_status" = "not_installed" ]; then
    echo "Warning: Lighthouse Validator service is not installed."
fi

echo "Replacing previous version..."

# Check if the file exists before trying to remove it
if [ -f /usr/local/bin/lighthouse ]; then
    sudo rm /usr/local/bin/lighthouse
fi

# Check if the source file exists before copying
if [ -f ~/.cargo/bin/lighthouse ]; then
    sudo cp ~/.cargo/bin/lighthouse /usr/local/bin
else
    echo "Error: Source file does not exist. Installation aborted."
    exit 1
fi

if [ "$beacon_status" = "active" ]; then
    echo "Restarting Lighthouse Beacon..."
    sudo systemctl start lighthousebeacon.service
fi

if [ "$validator_status" = "active" ]; then
    echo "Restarting Lighthouse Validator..."
    sudo systemctl start lighthousevalidator.service
fi
```
{% endcode %}

Make all scripts executable.

```bash
chmod u+x ~/lighthouse-build.sh
chmod u+x ~/lighthouse-deploy.sh
```
