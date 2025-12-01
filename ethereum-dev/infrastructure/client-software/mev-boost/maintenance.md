---
description: Notes on how to maintain and update an MEV Boost client.
icon: toolbox
---

# Maintenance

### MEV Boost - Update Client

```bash
mev-update
```

### MEV Boost - Update mevboost.service

```bash
mev-stop
mev-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
mev-start
mev-status
```
