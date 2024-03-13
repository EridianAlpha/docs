---
description: Notes on how to maintain and update a Lighthouse Beacon Node.
---

# 🏗️ Maintenance

### Lighthouse - Update lighthousebeacon.service

```bash
lighthouse-beacon-stop
lighthouse-beacon-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
lighthouse-beacon-start
lighthouse-beacon-status
```
