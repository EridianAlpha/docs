---
description: Notes on how to maintain and update a Lighthouse client.
---

# 🏗️ Maintenance

## Lighthouse - Update Client

```bash
lighthouse-version-current    # Check current version number
lighthouse-build              # Download and build latest version
lighthouse-version-new        # Check the version of the newly built client
lighthouse-deploy             # Deploy the new client
```

## Lighthouse - Update lighthousebeacon.service

```bash
lighthouse-beacon-stop
lighthouse-beacon-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
lighthouse-beacon-start
lighthouse-beacon-status
```
