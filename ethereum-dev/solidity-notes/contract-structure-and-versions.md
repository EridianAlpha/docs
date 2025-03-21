# Contract Structure & Versions

### Code License

* Specify a license at the top of every file.

```solidity
// SPDX-License-Identifier: MIT
```

### Solidity Versions

{% code fullWidth="false" %}
```solidity
pragma solidity 0.8.7;              // A specific version
pragma solidity ^0.8.7;             // Version 0.8.7 or higher
pragma solidity >=0.8.7 <0.9.0;     // Only versions including or greater than 0.8.7 
                                    // and less than 0.9.0
```
{% endcode %}

### Contract Template

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HelloWorld {
}
```

### Import

* When you have multiple files and you want to import one file into another, Solidity uses the `import`keyword:

```solidity
import "./someothercontract.sol";

contract newContract is SomeOtherContract {
}
```

* So if we had a file named `someothercontract.sol` in the same directory as this contract (that's what the `./`means), it would get imported by the compiler.
* Imports can also be renamed when being imported:

```solidity
import {WETH9Test as Handler} from "./WETH9Test.t.sol";

contract WETH9TestInvariant is Test, Counter {

    Handler public handler;

    function setUp() public {
        handler = new Handler();
    }
}
```

### Contract Layout

* license
* version
* imports
* errors
* interfaces, libraries, contracts
* type declarations
* state variables
* events
* modifiers
* functions

### Function Layout

* constructor
* receive function (if exists)
* fallback function (if exists)
* external
* public
* internal
* private
* view & pure functions
