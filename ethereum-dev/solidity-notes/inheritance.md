# Inheritance

* Rather than making one extremely long contract, sometimes it makes sense to split your code logic across multiple contracts to organize the code.
* One feature of Solidity that makes this more manageable is contract `inheritance`.
* Solidity supports multiple inheritance. Contracts can inherit other contract by using the `is` keyword.
* Function that is going to be overridden by a child contract must be declared as `virtual`.
* Function that is going to override a parent function must use the keyword `override`.
* Order of inheritance is important.
* You have to list the parent contracts in the order from “most base-like” to “most derived”.

```solidity
contract Doge {
  function catchphrase() public returns (string memory) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string memory) {
    return "Such Moon BabyDoge";
  }
}
```

`BabyDoge` inherits from `Doge`. That means if you compile and deploy `BabyDoge`, it will have access to both `catchphrase()`and `anotherCatchphrase()` (and any other public functions we may define on `Doge`).

This can be used for logical inheritance (such as with a subclass, a Cat is an Animal). But it can also be used simply for organizing your code by grouping similar logic together into different contracts.

### Example when constructor arguments are required

{% code fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import "./FundMe.sol";

contract FundMeMatching is FundMe {
    // Type declarations
    using PriceConverter for uint256; // Extends uint256 (used from msg.value) to enable direct price conversion

    /**
     * This is how to create a constructor for an inherited contract
     * if the parent already has a constructor that has arguments passed
     * https://docs.soliditylang.org/en/develop/contracts.html#arguments-for-base-constructors
     */
    constructor(address priceFeedAddress) FundMe(priceFeedAddress) {}

    function fund() public payable override {
        if (msg.value.getConversionRate(s_priceFeed) <= MINIMUM_USD)
            revert FundMe__NotEnoughEthSent();
        s_addressToAmountFunded[msg.sender] += msg.value;
        s_funders.push(msg.sender);
    }
}
```
{% endcode %}

### Example

* [https://solidity-by-example.org/inheritance/](https://solidity-by-example.org/inheritance/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

/* Graph of inheritance
    A
   / \
  B   C
 / \ /
F  D,E

*/

contract A {
    function foo() public pure virtual returns (string memory) {
        return "A";
    }
}

// Contracts inherit other contracts by using the keyword 'is'.
contract B is A {
    // Override A.foo()
    function foo() public pure virtual override returns (string memory) {
        return "B";
    }
}

contract C is A {
    // Override A.foo()
    function foo() public pure virtual override returns (string memory) {
        return "C";
    }
}

// Contracts can inherit from multiple parent contracts.
// When a function is called that is defined multiple times in
// different contracts, parent contracts are searched from
// right to left, and in depth-first manner.

contract D is B, C {
    // D.foo() returns "C"
    // since C is the right most parent contract with function foo()
    function foo() public pure override(B, C) returns (string memory) {
        return super.foo();
    }
}

contract E is C, B {
    // E.foo() returns "B"
    // since B is the right most parent contract with function foo()
    function foo() public pure override(C, B) returns (string memory) {
        return super.foo();
    }
}

// Inheritance must be ordered from “most base-like” to “most derived”.
// Swapping the order of A and B will throw a compilation error.
contract F is A, B {
    function foo() public pure override(A, B) returns (string memory) {
        return super.foo();
    }
}
```

### Inherited State Variables

* Unlike functions, state variables cannot be overridden by re-declaring it in the child contract.
* The only way it can be overridden is by setting it directly in the construction of the child contract.
  * [https://solidity-by-example.org/shadowing-inherited-state-variables/](https://solidity-by-example.org/shadowing-inherited-state-variables/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract A {
    string public name = "Contract A";

    function getName() public view returns (string memory) {
        return name;
    }
}

// Shadowing is disallowed in Solidity 0.6
// This will not compile
// contract B is A {
//     string public name = "Contract B";
// }

contract C is A {
    // This is the correct way to override inherited state variables.
    constructor() {
        name = "Contract C";
    }

    // C.getName returns "Contract C"
}
```



