---
description: Notes on how to maintain and update a Teku client.
icon: toolbox
---

# Maintenance

## Teku - Update Client

```bash
teku-version-current    # Check current version number
teku-build              # Download and build latest version
teku-version-new        # Check the version of the newly built client
teku-deploy             # Deploy the new client
```

## Teku - Update tekubeacon.service

```bash
teku-beacon-stop
teku-beacon-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
teku-beacon-start
teku-beacon-status
```
