---
description: Lighthouse Validator client installation guide.
---

# 💾 Installation

* [Create Aliases](installation.md#create-aliases)
* [Lighthouse VC - Configure Service](installation.md#lighthouse-vc-configure-service)

### Create Aliases

{% code fullWidth="true" %}
```bash
echo "alias lighthouse-validator-log='journalctl -f -u lighthousevalidator.service -o cat | ccze -A'" >> ~/.bashrc
echo "alias lighthouse-validator-start='sudo systemctl start lighthousevalidator.service'" >> ~/.bashrc
echo "alias lighthouse-validator-stop='sudo systemctl stop lighthousevalidator.service'" >> ~/.bashrc
echo "alias lighthouse-validator-restart='sudo systemctl restart lighthousevalidator.service'" >> ~/.bashrc
echo "alias lighthouse-validator-status='sudo systemctl status lighthousevalidator.service'" >> ~/.bashrc
echo "alias lighthouse-validator-config='sudo vim /etc/systemd/system/lighthousevalidator.service'" >> ~/.bashrc
echo "alias lighthouse-validator-enable='sudo systemctl enable lighthousevalidator.service'" >> ~/.bashrc
echo "alias lighthouse-validator-disable='sudo systemctl disable lighthousevalidator.service'" >> ~/.bashrc

source ~/.bashrc
```
{% endcode %}

## Lighthouse VC - Configure Service

Create `lighthousevalidator` user and set permissions.

```bash
sudo useradd --no-create-home --shell /bin/false lighthousevalidator
sudo mkdir -p /var/lib/lighthouse/validators
sudo chown -R lighthousevalidator:lighthousevalidator /var/lib/lighthouse/validators

# <THIS IS NEEDED SO THE VALIDATOR CAN IMPORT THE KEYSTORES>
sudo chmod 700 /var/lib/lighthouse/validators
```

Configure `lighthousevalidator` service using the command line flags.

```bash
sudo vim /etc/systemd/system/lighthousevalidator.service
```

{% tabs %}
{% tab title="/etc/systemd/system/lighthousevalidator.service" %}
{% code title="/etc/systemd/system/lighthousevalidator.service" %}
```bash
[Unit]
Description=Lighthouse Ethereum Client - Validator Client
After=network-online.target
Wants=network-online.target

[Service]
User=lighthousevalidator
Group=lighthousevalidator
Type=simple
Restart=always
RestartSec=5

Environment=NETWORK=                    # E.g. mainnet or holesky
Environment=DATADIR=                    # Default: /var/lib/lighthouse
Environment=GRAFFITI=                   # E.g. For the merge!
Environment=METRICS_PORT=               # Default: 5064
Environment=MONITORING_ENDPOINTS=
Environment=SUGGESTED_FEE_RECIPIENT=
Environment=BEACON_NODES=

ExecStart=/usr/local/bin/lighthouse vc \
    --network ${NETWORK} \
    --datadir ${DATADIR} \
    --graffiti ${GRAFFITI} \
    --metrics \
    --metrics-port ${METRICS_PORT} \
    --monitoring-endpoint ${MONITORING_ENDPOINTS} \
    --enable-doppelganger-protection \
    --suggested-fee-recipient ${SUGGESTED_FEE_RECIPIENT} \
    --builder-proposals \
    --beacon-nodes ${BEACON_NODES}

[Install]
WantedBy=multi-user.target
```
{% endcode %}
{% endtab %}

{% tab title="Lighthouse VC Flags Explained" %}
| `/usr/local/bin/lighthouse vc`     | Starts the validator node                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `--network`                        | <p>Name of the chain Lighthouse will sync and follow</p><ul><li>Possible values:</li><li><p></p><ul><li>mainnet</li><li>prater</li><li>gnosis</li><li>kiln</li><li>ropsten</li></ul></li></ul>                                                                                                                                                                                                                                                                                                                                                                                                         |
| `--datadir`                        | Used to specify a custom root data directory for lighthouse keys and databases                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `--graffiti`                       | Specify your custom graffiti to be included in blocks                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `--monitoring-endpoint`            | Set to a monitoring service e.g. [https://beaconcha.in](https://beaconcha.in/)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `--enable-doppelganger-protection` | <p>Protects from the chance of two instances of the validator starting on the same system</p><p><a href="https://lighthouse-book.sigmaprime.io/validator-doppelganger.html">https://lighthouse-book.sigmaprime.io/validator-doppelganger.html</a></p><p><br>Not perfect, and does have a penalty of waiting for 2-3 epochs (approx. 6.4 minutes per epoch)</p><p><br>It's a bit annoying to miss a few attestations, and would be very sad to miss a block, but the chance of me being a block proposer during that downtime is very low, and the benefits of not getting slashed are much greater</p> |
| `--metrics`                        | <p>Enable the Prometheus metrics HTTP server</p><ul><li>Disabled by default</li></ul>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `--suggested-fee-recipient`        | The fallback address provided to the `Beacon Node` if nothing suitable is found in the validator definitions or fee recipient file                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
{% endtab %}
{% endtabs %}

Start the service and check it's working as expected.

## Command Aliases

```bash
daemon-reload        # Reload any changes made to the lighthousevalidator.service
lighthouse-validator-enable     # Enable the lighthousevalidator.service
lighthouse-validator-start      # Start the lighthousevalidator.service
lighthouse-validator-status     # View the status of the lighthousevalidator.service

lighthouse-validator-log        # View the lighthousevalidator.service logs
```

At this point, the `lighthousevalidator.service` is running with no keystores.

So it will just show a message saying something like `No validators present` which is fine.
