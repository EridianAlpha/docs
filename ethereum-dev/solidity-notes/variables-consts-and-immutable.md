# Variables, Consts & Immutable

### Variables

There are 3 types of variables in Solidity:

* `local`
  * Declared inside a function.
  * Not stored on the blockchain.
* `state`
  * Declared outside a function.
  * Stored on the blockchain.
* `global`
  * Provides information about the blockchain.

### Example - Variables

* [https://solidity-by-example.org/variables/](https://solidity-by-example.org/variables/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract Variables {
    // State variables are stored on the blockchain.
    string public text = "Hello";
    uint public num = 123;

    function doSomething() public {
        // Local variables are not saved to the blockchain.
        uint i = 456;

        // Here are some global variables
        uint timestamp = block.timestamp; // Current block timestamp
        address sender = msg.sender; // address of the caller
    }
}
```

### Constants

* Constants are variables that cannot be modified
* Their value is hard coded into the bytecode of the contract
* Using constants can save gas cost

### Example - Constants

* [https://solidity-by-example.org/constants/](https://solidity-by-example.org/constants/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract Constants {
    // coding convention to uppercase constant variables
    address public constant MY_ADDRESS = 0x777788889999AaAAbBbbCcccddDdeeeEfFFfCcCc;
    uint public constant MY_UINT = 123;
}
```

### Immutable

* Immutable variables are like constants
* Values of immutable variables can be set inside the `constructor` but cannot be modified afterwards

### Example - Immutable

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract Immutable {
    // coding convention to uppercase constant variables
    address public immutable MY_ADDRESS;
    uint public immutable MY_UINT;

    constructor(uint _myUint) {
        MY_ADDRESS = msg.sender;
        MY_UINT = _myUint;
    }
}
```
