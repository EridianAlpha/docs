---
icon: weight-hanging
---

# Anvil

[https://book.getfoundry.sh/reference/anvil](https://book.getfoundry.sh/reference/anvil/)

* Start a local blockchain using `anvil`

```bash
anvil
```

* Set a block time in seconds: `--block-time 12`
* Set a port (`http` and `ws` use the same port): `--port 8440`

### Forked Chains

```
anvil --fork-url <CHAIN_EL_RPC> --steps-tracing --chain-id <CHAIN_ID>
```
