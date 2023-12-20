# Receive & Fallback

```solidity
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
```

## Fallback

* `fallback` is a function that does not take any arguments and does not return anything.
* It is executed either when:
  * A function that does not exist is called
  * Ether is sent directly to a contract but `receive()` does not exist or `msg.data` is not empty
* `fallback` has a 2300 gas limit when called by `transfer` or `send`

### Fallback Example

* [https://solidity-by-example.org/fallback/](https://solidity-by-example.org/fallback/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {
    event Log(uint gas);

    // Fallback function must be declared as external.
    fallback() external payable {
        // send / transfer (forwards 2300 gas to this fallback function)
        // call (forwards all of the gas)
        emit Log(gasleft());
    }

    // Helper function to check the balance of this contract
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}

contract SendToFallback {
    function transferToFallback(address payable _to) public payable {
        _to.transfer(msg.value);
    }

    function callFallback(address payable _to) public payable {
        (bool sent, ) = _to.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
    }
}
```

## Receive

The `receive` function in Solidity is a special type of function that is triggered when ETH is sent to a contract and `msg.data` is empty. It's a newer addition to Solidity, designed to make contracts more intuitive and safer. This function is executed in the following scenarios:

When ETH is sent to the contract with an empty `msg.data`.

* If `receive()` exists, and `msg.data` is empty, this function is invoked.
* If `receive()` does not exist, but Ether is sent with empty `msg.data`, the `fallback` function is used instead.

This function is specified with the `receive() external payable` declaration, indicating that it can receive ETH. The use of `receive()` makes the intentions of a contract regarding Ether transfers more explicit, which is crucial for contract security and functionality.

### Receive Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ReceiveExample {
    event Received(address sender, uint amount);

    // The receive function is triggered when Ether is sent to the contract.
    receive() external payable {
        emit Received(msg.sender, msg.value);
    }

    // Helper function to check the balance of this contract
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}

```

In this example, when Ether is sent to the `ReceiveExample` contract with empty `msg.data`, the `receive()` function is triggered, and it emits an event logging the sender's address and the amount of Ether sent. The `getBalance` function is a helper that allows querying the contract's balance.
