# 🧪 Tests

{% hint style="info" %}
**Quick Notes**


{% endhint %}



* [https://book.getfoundry.sh/forge/writing-tests](https://book.getfoundry.sh/forge/writing-tests)

Tests are written in Solidity. If the test function reverts, the test fails, otherwise it passes.

<table><thead><tr><th width="206">Test</th><th>Description</th></tr></thead><tbody><tr><td>Unit</td><td>Testing a specific part of our code</td></tr><tr><td>Integration</td><td>Testing how our code works with other parts of our code</td></tr><tr><td>Forked</td><td>Testing our code on a simulated real environment</td></tr><tr><td>Staging</td><td>Testing our code in a real environment that is not prod</td></tr><tr><td>Fuzz</td><td>Testing our code against unexpected/random inputs to find bugs</td></tr></tbody></table>

### Writing Tests

DSTest provides basic logging and assertion functionality. To get access to the functions, import `forge-std/Test.sol` and inherit from `Test` in your test contract:

```solidity
import "forge-std/Test.sol";
```

{% code title="Example Test Contract" %}
```solidity
pragma solidity 0.8.10;

import "forge-std/Test.sol";

contract ContractBTest is Test {
    uint256 testNumber;

    // An optional function invoked before each test case is run.
    function setUp() public {
        testNumber = 42;
    }

    // Functions prefixed with test are run as a test case.
    function test_NumberIs42() public {
        assertEq(testNumber, 42);
    }

    // Inverse test prefix - if the function does not revert, the test fails.
    function testFail_Subtract43() public {
        testNumber -= 43;
    }
}

```
{% endcode %}

{% hint style="info" %}
Tests are deployed to `0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84`. If you deploy a contract within your test, then `0xb4c...7e84` will be its deployer. If the contract deployed within a test gives special permissions to its deployer, such as `Ownable.sol`'s `onlyOwner` modifier, then the test contract `0xb4c...7e84` will have those permissions.
{% endhint %}

{% hint style="warning" %}
Test functions must have either `external` or `public` visibility. Functions declared as `internal` or `private` won't be picked up by Forge, even if they are prefixed with `test`.
{% endhint %}

### Shared test setups

* If there are multiple tests that all have the same  initial setup configuration, use a helper contract to reduce code duplication

```solidity
abstract contract TestHelperContract is Test, {
    address constant IMPORTANT_ADDRESS = 0x543d...;
    SomeContract someContract;
    constructor() {...}
}

contract MyContractTest is TestHelperContract {
    function setUp() public {
        someContract = new SomeContract(0, IMPORTANT_ADDRESS);
        ...
    }
}

contract MyOtherContractTest is TestHelperContract {
    function setUp() public {
        someContract = new SomeContract(1000, IMPORTANT_ADDRESS);
        ...
    }
}

```

### Running tests

```bash
forge test --summary
forge test -vvv
forge test -m <SPECIFIC_TEST_FUNCTION_NAME>
```

#### Unit Testing

TODO

#### Integration Testing

TODO

#### Forked Testing

TODO

#### Staging Testing

TODO

#### Fuzz Testing

TODO: Turn this video into notes.

{% embed url="https://www.youtube.com/watch?v=juyY-CTolac" %}



### Test Coverage

```bash
forge coverage
```



#### Coverage line highlighting

* [https://mirror.xyz/devanon.eth/RrDvKPnlD-pmpuW7hQeR5wWdVjklrpOgPCOA-PJkWFU](https://mirror.xyz/devanon.eth/RrDvKPnlD-pmpuW7hQeR5wWdVjklrpOgPCOA-PJkWFU)

```bash
forge coverage --report lcov
```

* Open the command palette in VS Code (`CMD+SHIFT+P` or `CTRL+SHIFT+P` by default) and type “Display Coverage”







