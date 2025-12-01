---
description: Notes on how to use a Geth Client.
icon: code
---

# Useful Commands

{% hint style="info" %}
All of the alias commands have been defined as aliases in \~/`.bashrc when`installing Erigon.
{% endhint %}

### erigon.service

```bash
erigon-log            # View the erigon.service logs
erigon-start          # Start the erigon.service
erigon-stop           # Stop the erigon.service
erigon-restart        # Restart the erigon.service
erigon-status         # View the status of the erigon.service
erigon-version        # Check the version of Erigon in use
erigon-enable         # Enable the erigon.service
erigon-disable        # Disable the erigon.service
erigon-delete-data    # Delete all Erigon chain data

erigon-config         # Open the /etc/systemd/system/erigon.service in vim
daemon-reload         # Reload any changes made to the erigon.service
```

### Data Locations

`Erigon` `chaindata` location.

```
/var/lib/erigon/chaindata
```
