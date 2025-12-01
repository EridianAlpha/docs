---
description: Notes on how to maintain and update a Besu Client.
icon: toolbox
---

# Maintenance

### Besu - Update Client

```bash
besu-update
```

### Besu - Update besu.service

```bash
besu-stop
besu-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
besu-start
besu-status
```
