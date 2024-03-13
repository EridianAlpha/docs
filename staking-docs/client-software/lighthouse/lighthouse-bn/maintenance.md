---
description: Notes on how to maintain and update a Lighthouse Beacon Node.
---

# 🏗️ Maintenance

### Lighthouse - Update lighthousebeacon.service

{% tabs %}
{% tab title="Command Aliases" %}
```bash
lighthouse-beacon-stop
lighthouse-beacon-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
lighthouse-beacon-start
lighthouse-beacon-status
```
{% endtab %}

{% tab title="Full Commands" %}
```bash
sudo systemctl stop lighthousebeacon.service
sudo vim /etc/systemd/system/lighthousebeacon.service

# MAKE ANY CHANGES TO THE CONFIG

sudo systemctl daemon-reload
sudo systemctl start lighthousebeacon.service
sudo systemctl status lighthousebeacon.service
```
{% endtab %}
{% endtabs %}
