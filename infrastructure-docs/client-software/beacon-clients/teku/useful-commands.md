---
description: Notes on how to use a Teku BN.
---

# ⌨️ Useful Commands

### tekubeacon.service

{% code fullWidth="false" %}
```bash
teku-beacon-log            # View the tekubeacon.service logs
teku-beacon-start          # Start the tekubeacon.service
teku-beacon-stop           # Stop the tekubeacon.service
teku-beacon-restart        # Restart the tekubeacon.service
teku-beacon-status         # View the status of the tekubeacon.service
teku-beacon-enable         # Enable the tekubeacon.service
teku-beacon-disable        # Disable the tekubeacon.service
teku-beacon-delete-data    # Delete all Teku chain data

teku-beacon-config         # Open the /etc/systemd/system/tekubeacon.service in vim
daemon-reload              # Reload any changes made to the tekubeacon.service
```
{% endcode %}
