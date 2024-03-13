---
description: Notes on how to use a Lighthouse Beacon Node.
---

# ⌨️ Useful commands

### lighthousebeacon.service

{% tabs %}
{% tab title="Command Aliases" %}
```bash
lighthouse-beacon-log        # View the lighthousebeacon.service logs
lighthouse-beacon-start      # Start the lighthousebeacon.service
lighthouse-beacon-stop       # Stop the lighthousebeacon.service
lighthouse-beacon-restart    # Restart the lighthousebeacon.service
lighthouse-beacon-status     # View the status of the lighthousebeacon.service
lighthouse-beacon-enable     # Enable the lighthousebeacon.service
lighthouse-beacon-disable    # Disable the lighthousebeacon.service

lighthouse-beacon-config     # Open the /etc/systemd/system/lighthousebeacon.service in vim
daemon-reload                # Reload any changes made to the lighthousebeacon.service
```
{% endtab %}

{% tab title="Full Commands" %}
```bash
sudo journalctl -f -u lighthousebeacon.service -o cat | ccze -A   # View the lighthousebeacon.service logs
sudo systemctl start lighthousebeacon.service                     # Start the lighthousebeacon.service
sudo systemctl stop lighthousebeacon.service                      # Stop the lighthousebeacon.service
sudo systemctl restart lighthousebeacon.service                   # Restart the lighthousebeacon.service
sudo systemctl status lighthousebeacon.service                    # View the status of the lighthousebeacon.service
sudo systemctl enable lighthousebeacon.service                    # Enable the lighthousebeacon.service
sudo systemctl disable lighthousebeacon.service                   # Disable the lighthousebeacon.service

sudo vim /etc/systemd/system/lighthousebeacon.service             # Open the /etc/systemd/system/lighthousebeacon.service in vim
sudo systemctl daemon-reload                                      # Reload any changes made to the lighthousebeacon.service
```
{% endtab %}
{% endtabs %}
