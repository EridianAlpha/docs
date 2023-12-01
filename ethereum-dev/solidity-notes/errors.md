# Errors

* An error will undo all changes made to the state during a transaction
* You can throw an error by calling `require`, `revert` or `assert`

### require

* Used to validate inputs and conditions before execution.

```solidity
// Used in both test contracts and main contracts
require(success, "Call failed"); 
```

### revert

* Similar to require, revert is useful when the condition to check is complex.
* So put it at the end of a longer logic statement making it easier to read.

```solidity
// This error definition is needed in the main contract and the test contract
error FundMe__RefundFailed();

// In main contract
if (!callSuccess) revert FundMe__RefundFailed();

// In test contract
// Expects the next line to revert with the specified error
vm.expectRevert(FundMe__RefundFailed.selector);
testHelper.fundMeRefund();
```

### assert

```solidity
// Used in both test contracts and main contracts
assert(funders.length == 3);
assertEq(funders.length, 3); // More informative logs
```

### Gas Saving

```solidity
// Uses more gas as a custom error is stored as a string (bytes array)
require(msg.sender == i_owner);

// Uses less gas as the error is not stored as a string
if (msg.sender != i_owner) revert FundMe__NotOwner();
```

### Example

* [https://solidity-by-example.org/error/](https://solidity-by-example.org/error/)

{% code title="Example 1" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Error {
    function testRequire(uint _i) public pure {
        // Require should be used to validate conditions such as:
        // - inputs
        // - conditions before execution
        // - return values from calls to other functions
        require(_i > 10, "Input must be greater than 10");
    }

    function testRevert(uint _i) public pure {
        // Revert is useful when the condition to check is complex.
        // This code does the exact same thing as the example above
        if (_i <= 10) {
            revert("Input must be greater than 10");
        }
    }

    uint public num;

    function testAssert() public view {
        // Assert should only be used to test for internal errors,
        // and to check invariants.

        // Here we assert that num is always equal to 0
        // since it is impossible to update the value of num
        assert(num == 0);
    }

    // custom error
    error InsufficientBalance(uint balance, uint withdrawAmount);

    function testCustomError(uint _withdrawAmount) public view {
        uint bal = address(this).balance;
        if (bal < _withdrawAmount) {
            revert InsufficientBalance({balance: bal, withdrawAmount: _withdrawAmount});
        }
    }
}
```
{% endcode %}

{% code title="Example 2" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Account {
    uint public balance;
    uint public constant MAX_UINT = 2**256 - 1;

    function deposit(uint _amount) public {
        uint oldBalance = balance;
        uint newBalance = balance + _amount;

        // balance + _amount does not overflow if balance + _amount >= balance
        require(newBalance >= oldBalance, "Overflow");

        balance = newBalance;

        assert(balance >= oldBalance);
    }

    function withdraw(uint _amount) public {
        uint oldBalance = balance;

        // balance - _amount does not underflow if balance >= _amount
        require(balance >= _amount, "Underflow");

        if (balance < _amount) {
            revert("Underflow");
        }

        balance -= _amount;

        assert(balance <= oldBalance);
    }
}
```
{% endcode %}
