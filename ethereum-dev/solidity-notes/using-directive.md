# Using Directive

{% embed url="https://docs.soliditylang.org/en/v0.8.26/grammar.html#a4.SolidityParser.usingDirective" %}

{% hint style="warning" %}
Not inherited by child contracts. It must be imported and defined in every contract in which the "Using Directive" is used.
{% endhint %}

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {Address} from "@openzeppelin/contracts/utils/Address.sol";

contract UsingDirective {
    using Address for address payable;

    function sendETH(address _to) public payable {
        payable(address(_to)).sendValue(msg.value);
    }
}
```
