# 🤝 Useful Commands

### Run Scripts

```
forge build
forge script script/<SCRIPT>:<CONTRACT_NAME> --rpc-url http://<RPC_URL>
forge script script/<SCRIPT>:<CONTRACT_NAME> --fork-url http://<RPC_URL>
```

* Foundry DevOps
  * A repo to get the most recent deployment from a given environment in Foundry. This way, you can do scripting off previous deployments in solidity.
  * [https://github.com/Cyfrin/foundry-devops](https://github.com/Cyfrin/foundry-devops)

```bash
forge install Cyfrin/foundry-devops --no-commit
```

```solidity
import {DevOpsTools} from "lib/foundry-devops/src/DevOpsTools.sol";
import {MyContract} from "my-contract/MyContract.sol";

function interactWithPreviouslyDeployedContracts() public {
    address contractAddress = DevOpsTools.get_most_recent_deployment("MyContract", block.chainid);
    MyContract myContract = MyContract(contractAddress);
    myContract.doSomething();
}
```

### Makefile

* `@` stops the command being printed out to the command line which is useful when you don't want sensitive info to be printed e.g. private keys.

{% tabs %}
{% tab title="FundMe Usage" %}
{% code overflow="wrap" %}
```makefile
-include .env

build:; forge build # ; is used to run multiple commands in one line

deploy-holesky:
	forge script script/DeployFundMe.s.sol:DeployFundMe --rpc-url $(HOLESKY_RPC_URL) --private-key $(HOLESKY_PRIVATE_KEY) --broadcast -vvvv

deploy-holesky-verify:
	forge script script/DeployFundMe.s.sol:DeployFundMe --rpc-url $(HOLESKY_RPC_URL) --private-key $(HOLESKY_PRIVATE_KEY) --broadcast --verify --etherscan-api-key $(ETHERSCAN_API_KEY) -vvvv
```
{% endcode %}
{% endtab %}

{% tab title="Full example" %}
```makefile
-include .env

.PHONY: all test clean deploy fund help install snapshot format anvil 

DEFAULT_ANVIL_KEY := 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

help:
	@echo "Usage:"
	@echo "  make deploy [ARGS=...]\n    example: make deploy ARGS=\"--network sepolia\""
	@echo ""
	@echo "  make fund [ARGS=...]\n    example: make deploy ARGS=\"--network sepolia\""

all: clean remove install update build

# Clean the repo
clean  :; forge clean

# Remove modules
remove :; rm -rf .gitmodules && rm -rf .git/modules/* && rm -rf lib && touch .gitmodules && git add . && git commit -m "modules"

install :; forge install cyfrin/foundry-devops@0.0.11 --no-commit && forge install smartcontractkit/chainlink-brownie-contracts@0.6.1 --no-commit && forge install foundry-rs/forge-std@v1.5.3 --no-commit

# Update Dependencies
update:; forge update

build:; forge build

test :; forge test 

snapshot :; forge snapshot

format :; forge fmt

anvil :; anvil -m 'test test test test test test test test test test test junk' --steps-tracing --block-time 1

NETWORK_ARGS := --rpc-url http://localhost:8545 --private-key $(DEFAULT_ANVIL_KEY) --broadcast

ifeq ($(findstring --network sepolia,$(ARGS)),--network sepolia)
	NETWORK_ARGS := --rpc-url $(SEPOLIA_RPC_URL) --private-key $(PRIVATE_KEY) --broadcast --verify --etherscan-api-key $(ETHERSCAN_API_KEY) -vvvv
endif

deploy:
	@forge script script/DeployFundMe.s.sol:DeployFundMe $(NETWORK_ARGS)

fund:
	@forge script script/Interactions.s.sol:FundFundMe $(NETWORK_ARGS)

withdraw:
	@forge script script/Interactions.s.sol:WithdrawFundMe $(NETWORK_ARGS)
```
{% endtab %}
{% endtabs %}

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

{% hint style="info" %}
If you see the error `EvmError: NotActivated` when trying to deploy contracts using `forge` it may be that you need to explicitly set the `--evm-version` when running the forge command
{% endhint %}

### Inspect

* List out all the methods and function selectors in a contract

```bash
forge inspect <CONTRACT_NAME> methods
```
