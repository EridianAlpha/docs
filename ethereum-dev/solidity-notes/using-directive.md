# Using Directive

{% embed url="https://docs.soliditylang.org/en/v0.8.26/grammar.html#a4.SolidityParser.usingDirective" %}

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

// Library Directive Imports
import {Address} from "@openzeppelin/contracts/utils/Address.sol";

contract SimpleSend {
    using Address for address payable;
    
    function sendETH(uint256 value) public {
        payable(0x25941dC771bB64514Fc8abBce970307Fb9d477e9).sendValue(value);
    }
}
```
