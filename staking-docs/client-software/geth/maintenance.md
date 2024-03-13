---
description: Notes on how to maintain and update a Geth Client.
---

# 🏗️ Maintenance

* [Go - Update](maintenance.md#go-update)
* [Geth - Update Client](maintenance.md#geth-update-client)
* [Geth - Update geth.service](maintenance.md#geth-update-geth.service)
* [Geth - Rollback Chain to Previous Block Number](maintenance.md#geth-rollback-chain-to-previous-block-number)
* [Geth - Resync after an Unexpected Shutdown](maintenance.md#geth-resync-after-an-unexpected-shutdown)
* [Geth - Pruning](maintenance.md#geth-pruning)

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

### Geth - Update Client

```bash
geth-update
```

### Geth - Update geth.service

```bash
geth-stop
geth-config

# MAKE ANY CHANGES TO THE CONFIG

daemon-reload
geth-start
geth-status
```

### Geth - Rollback Chain to Previous Block Number

This was needed for a bug introduced in Geth v.1.10.22 that required a rollback to a previous block

Add `debug` flag to `--http.api`

```bash
geth-stop
geth-config

# Add "debug" to http.api
# --http.api="engine,eth,web3,net,debug"

daemon-reload
geth-start
geth-attach
```

In the `Geth` console set the new block head e.g. `debug.setHead("0xEAC1A8")`.

```javascript
debug.setHead("0x<BLOCK_NUMBER_IN_HEX>")
```

Once re-sync has been completed, go back and remove the `debug` flag from the `--http.api` argument.

### Geth - Resync after an Unexpected Shutdown

To avoid duplication these details can be found on the EthStaker Knowledge Base.

* [How to resync Geth](https://ethstaker.gitbook.io/ethstaker-knowledge-base/tutorials/resync-geth)

### Geth - Pruning

```bash
geth-stop

tmux new -s prune-geth
sudo /usr/local/bin/geth --datadir /var/lib/goethereum snapshot prune-state

exit
geth-start
```
