---
description: Notes on how to maintain and update a Lighthouse Validator Client.
icon: toolbox
---

# Maintenance

### Lighthouse - Update lighthousevalidator.service

```bash
lighthouse-validator-stop
lighthouse-validator-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
lighthouse-validator-start
lighthouse-validator-status
```
