# 🔥 Forge

### Inspect

{% code title="List out all the methods and function selectors in a contract" %}
```bash
forge inspect <CONTRACT_NAME> methods
```
{% endcode %}

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
If you see the error `EvmError: NotActivated` when trying to deploy contracts using `forge` it may be that you need to explicitly set the `--evm-version` when running the forge command.

The evm version can also be set in the `foundry.toml` file:

```toml
[profile.default]
evm_version = "cancun"
```
{% endhint %}

### Contract Verification

If automatic contract verification fails during deployment it can be performed manually using forge.

{% code overflow="wrap" %}
```bash
forge verify-contract <CONTRACT_ADDRESS> ./src/<CONTRACT>.sol:<CONTRACT_NAME> --chain-id <CHAIN_ID> --watch --etherscan-api-key <ETHERSCAN_API_KEY>
```
{% endcode %}

* `--watch` leaves the terminal waiting for a status confirmation reply.
* If constructor arguments are required they can be encoded using `cast` e.g for a constructor requiring a single address:

```solidity
cast abi-encode "constructor(address)" <ADDRESS_TO_PASS_INTO_CONSTRUCTOR>
```

* Then use that `cast` output  to add `--constructor-args` to the end of the `forge verify-contract` command.

{% hint style="info" %}
For Ethereum L1 or L1 testnets use the Etherscan API key: [https://etherscan.io/myapikey](https://etherscan.io/myapikey)

For L2s (e.g. Base) or L2 testnets use a Basescan API key (but still call it `--etherscan-api-key` in the command): [https://basescan.org/myapikey](https://basescan.org/myapikey)
{% endhint %}
