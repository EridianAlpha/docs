# 🤝 Useful Commands

### Run Scripts

```
forge build
forge script script/<SCRIPT>
```

### Anvil

* Start a local blockchain using `anvil`

```bash
anvil
```

### Cast

{% code title="Hex Converstion" %}
```bash
cast --to-base 0x6f dec
cast --to-hex 111
```
{% endcode %}

{% code title="Get storage slot from any contract" %}
```bash
cast storage <CONTRACT_ADDRRESS> <STORAGE_SLOT> --rpc-url <RPC_URL>
```
{% endcode %}

{% code title="Get OPCODES for a contract" %}
```bash
CONTRACT_NAME=

BYTECODE=$(jq -r '.bytecode.object' out/${CONTRACT_NAME}.sol/${CONTRACT_NAME}.json)
cast disassemble $BYTECODE > "${CONTRACT_NAME}-opcodes.txt"
```
{% endcode %}

### Forge

{% code title="Inspect contract and view storage layout" %}
```bash
forge inspect <CONTRACT_NAME> storageLayout
```
{% endcode %}
