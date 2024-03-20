---
description: Notes on how to use a Besu Client.
---

# ⌨️ Useful Commands

{% hint style="info" %}
All of the alias commands have been defined as aliases in \~/`.bashrc when`installing Besu.
{% endhint %}

### geth.service

```bash
besu-log            # View the besu.service logs
besu-start          # Start the besu.service
besu-stop           # Stop the besu.service
besu-restart        # Restart the besu.service
besu-status         # View the status of the besu.service
besu-version        # Check the version of Besu in use
besu-enable         # Enable the besu.service
besu-disable        # Disable the besu.service
besu-delete-data    # Delete all Besu chain data

besu-config         # Open the /etc/systemd/system/geth.service in vim
daemon-reload       # Reload any changes made to the geth.service
```

### Other Useful Commands

Checks `Besu` is running.

```bash
BESU_PORT=        # Default 8545

curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}' http://localhost:${GETH_PORT}
```

Check ports can be accessed.

```bash
PORTS_TO_CHECK=        # E.g. 30303,9000

curl https://eth2-client-port-checker.vercel.app/api/checker?ports=${PORTS_TO_CHECK}
```

Checks the validator status using the validator's public address.

```bash
VALIDATOR_HTTP_PORT=        # Lighthouse default: 5052
VALIDATOR_PUBLIC_KEY=       # E.g. 0x88426dd9c3d2bb71ad862a2e47d537304de528e88f5164f6db6ec423f1f7ed24d050c27ae4df45b37d2a4931fc820edf

curl -s http://127.0.0.1:${VALIDATOR_HTTP_PORT}/eth/v1/beacon/states/head/validators/${VALIDATOR_PUBLIC_KEY} |jq
```

Check validator status using a public endpoint in a browser.

```html
https://beaconstate.info/eth/v1/beacon/states/head/validators/<VALIDATOR_PUBLIC_KEY>
```

### Data Locations

`Besu` `chaindata` location.

```
/var/lib/besu/??
```
