# Send ETH (transfer, send, call)

### How to send Ether

You can send Ether to other contracts by

* `transfer` (2300 gas, throws error)
* `send` (2300 gas, returns bool)
* `call` (forward all gas or set gas, returns bool)

### How to receive Ether

A contract receiving Ether must have at least one of the functions below

* `receive()` external payable
* `fallback()` external payable
* `receive()` is called if `msg.data` is empty, otherwise `fallback()` is called.

### Which method should you use?

* `call` in combination with re-entrancy guard is the recommended method to use after December 2019.

### Guard against re-entrancy

* Making all state changes before calling other contracts.
* Using re-entrancy guard modifier.

### Example

* [https://solidity-by-example.org/sending-ether/](https://solidity-by-example.org/sending-ether/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract ReceiveEther {
    /*
    Which function is called, fallback() or receive()?

           send Ether
               |
         msg.data is empty?
              / \
            yes  no
            /     \
receive() exists?  fallback()
         /   \
        yes   no
        /      \
    receive()   fallback()
    */

    // Function to receive Ether. msg.data must be empty
    receive() external payable {}

    // Fallback function is called when msg.data is not empty
    fallback() external payable {}

    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}

contract SendEther {
    function sendViaTransfer(address payable _to) public payable {
        // This function is no longer recommended for sending Ether.
        _to.transfer(msg.value);
    }

    function sendViaSend(address payable _to) public payable {
        // Send returns a boolean value indicating success or failure.
        // This function is not recommended for sending Ether.
        bool sent = _to.send(msg.value);
        require(sent, "Failed to send Ether");
    }

    function sendViaCall(address payable _to) public payable {
        // Call returns a boolean value indicating success or failure.
        // This is the current recommended method to use.
        (bool sent, bytes memory data) = _to.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
    }
}
```





### Example - Call

* `call` is a low level function to interact with other contracts
* This is the recommended method to use when you're just sending Ether via calling the `fallback` function
* However it is not the recommend way to call existing functions
* [https://solidity-by-example.org/call/](https://solidity-by-example.org/call/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract Receiver {
    event Received(address caller, uint amount, string message);

    fallback() external payable {
        emit Received(msg.sender, msg.value, "Fallback was called");
    }

    function foo(string memory _message, uint _x) public payable returns (uint) {
        emit Received(msg.sender, msg.value, _message);

        return _x + 1;
    }
}

contract Caller {
    event Response(bool success, bytes data);

    // Let's imagine that contract B does not have the source code for
    // contract A, but we do know the address of A and the function to call.
    function testCallFoo(address payable _addr) public payable {
        // You can send ether and specify a custom gas amount
        (bool success, bytes memory data) = _addr.call{value: msg.value, gas: 5000}(
            abi.encodeWithSignature("foo(string,uint256)", "call foo", 123)
        );

        emit Response(success, data);
    }

    // Calling a function that does not exist triggers the fallback function.
    function testCallDoesNotExist(address _addr) public {
        (bool success, bytes memory data) = _addr.call(
            abi.encodeWithSignature("doesNotExist()")
        );

        emit Response(success, data);
    }
}
```



### Call - Specify Function

* [https://kushgoyal.com/ethereum-solidity-how-use-call-delegatecall/](https://kushgoyal.com/ethereum-solidity-how-use-call-delegatecall/)

{% code fullWidth="true" %}
```solidity
function myFunction(uint _x, address _addr) public returns(uint, uint) {
    // do something
    return (a, b);
}

// function signature string should not have any spaces
// 10 is the first parameter _x
// msg.sender is the second parameter _addr
(bool success, bytes memory result) = addr.call(abi.encodeWithSignature("myFunction(uint,address)", 10, msg.sender));
```
{% endcode %}

### Delegatecall

* `delegatecall` is a low level function similar to `call`
* `delegatecall` syntax is exactly the same as `call` syntax except it cannot accept the `value` option but only `gas`
* When contract A executes `delegatecall` to contract B, B's code is executed
* with contract A's storage, `msg.sender` and `msg.value`
* A popular and very useful use case for `delegatecall` is upgradable contracts
  * Upgradable contracts use a proxy contract which forwards all the function calls to the implementation contract using `delegatecall`
  * The address of the proxy contract remains constant while new implementations can be deployed multiple times
  * The address of the new implementation gets updated in the proxy contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

// NOTE: Deploy this contract first
contract B {
    // NOTE: storage layout must be the same as contract A
    uint public num;
    address public sender;
    uint public value;

    function setVars(uint _num) public payable {
        num = _num;
        sender = msg.sender;
        value = msg.value;
    }
}

contract A {
    uint public num;
    address public sender;
    uint public value;

    function setVars(address _contract, uint _num) public payable {
        // A's storage is set, B is not modified.
        (bool success, bytes memory data) = _contract.delegatecall(
            abi.encodeWithSignature("setVars(uint256)", _num)
        );
    }
}
```
