---
description: Notes on how to maintain and update a Geth Client.
---

# 🏗️ Maintenance

### Go - Update

Find the latest version of `Go` here: [https://go.dev/doc/install](https://go.dev/doc/install)

```bash
GO_LATEST_VERSION=    # Add the latest Go version here

cd ~/
wget https://go.dev/dl/go${GO_LATEST_VERSION}.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go${GO_LATEST_VERSION}.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
echo 'PATH="$PATH:/usr/local/go/bin"' >> ~/.profile
```

### Erigon - Update Client

```bash
erigon-update
```

### Erigon - Update erigon.service

```bash
erigon-stop
erigon-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
erigon-start
erigon-status
```
