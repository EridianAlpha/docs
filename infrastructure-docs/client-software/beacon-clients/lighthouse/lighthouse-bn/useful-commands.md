---
description: Notes on how to use a Lighthouse Beacon Node.
---

# ⌨️ Useful Commands

### lighthousebeacon.service

{% code fullWidth="true" %}
```bash
lighthouse-beacon-log            # View the lighthousebeacon.service logs
lighthouse-beacon-start          # Start the lighthousebeacon.service
lighthouse-beacon-stop           # Stop the lighthousebeacon.service
lighthouse-beacon-restart        # Restart the lighthousebeacon.service
lighthouse-beacon-status         # View the status of the lighthousebeacon.service
lighthouse-beacon-enable         # Enable the lighthousebeacon.service
lighthouse-beacon-disable        # Disable the lighthousebeacon.service
lighthouse-beacon-delete-data    # Delete all Lighthouse chain data

lighthouse-beacon-config         # Open the /etc/systemd/system/lighthousebeacon.service in vim
daemon-reload                    # Reload any changes made to the lighthousebeacon.service
```
{% endcode %}
