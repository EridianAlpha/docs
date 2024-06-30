# 🪄 Cast

* `call` is used for viewing data
* `send` is used for modifying state

### Send ETH to an address

{% code overflow="wrap" %}
```bash
cast send <SENDER_ADDRESS> --value 1ether --rpc-url ${RPC_URL} --private-key ${PRIVATE_KEY}
```
{% endcode %}

### Call a view function

{% code overflow="wrap" %}
```bash
cast call <CONTRACT_ADDRESS> "getCreator()" --rpc-url ${RPC_URL}
```
{% endcode %}

### Get and send tokens on an anvil fork

```bash
cast rpc anvil_impersonateAccount <TOKEN_WHALE> --rpc-url ${RPC_URL}
cast send <TOKEN_ADDRESS> \
--from <TOKEN_WHALE> \
"transfer(address,uint256)(bool)" \
<RECEIVER_ACCOUNT> \
10000000 \
--unlocked \
--rpc-url ${RPC_URL}
```

### Hex Conversion

```bash
cast --to-base 0x6f dec
cast --to-hex 111
```

### Get storage slot from any contract

```bash
cast storage <CONTRACT_ADDRRESS> <STORAGE_SLOT> --rpc-url <RPC_URL>
```

### Get OPCODES for a contract

```bash
CONTRACT_NAME=

BYTECODE=$(jq -r '.bytecode.object' out/${CONTRACT_NAME}.sol/${CONTRACT_NAME}.json)
cast disassemble $BYTECODE > "${CONTRACT_NAME}-opcodes.txt"
```
