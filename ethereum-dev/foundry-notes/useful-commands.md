# 🤝 Useful Commands

### Run Scripts

```
forge build
forge script script/<SCRIPT>
```

* Foundry DevOps
  * A repo to get the most recent deployment from a given environment in foundry. This way, you can do scripting off previous deployments in solidity.
  * [https://github.com/Cyfrin/foundry-devops](https://github.com/Cyfrin/foundry-devops)

```bash
forge install Cyfrin/foundry-devops --no-commit
```

```solidity
import {DevOpsTools} from "lib/foundry-devops/src/DevOpsTools.sol";
import {MyContract} from "my-contract/MyContract.sol";
.
.
.
function interactWithPreviouslyDeployedContracts() public {
    address contractAddress = DevOpsTools.get_most_recent_deployment("MyContract", block.chainid);
    MyContract myContract = MyContract(contractAddress);
    myContract.doSomething();
}
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

{% code title="Create .gas-snapshot for all tests" %}
```bash
forge snapshot
```
{% endcode %}
