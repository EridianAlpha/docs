# Constructor

* A constructor is an optional function that is executed upon contract creation.

{% hint style="info" %}
No functions, including the `receive()` function, can be invoked from logic in the constructor. Only logic and calls inside the constructor will execute as expected.

An unexpected consequence of this is if a call is made in the constructor that results in ETH being sent to the `receive()` function, as the ETH does get received, but the logic in the `receive()` function doesn't execute.
{% endhint %}

### Example

* [https://solidity-by-example.org/constructor/](https://solidity-by-example.org/constructor/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Base contract X
contract X {
    string public name;

    constructor(string memory _name) {
        name = _name;
    }
}

// Base contract Y
contract Y {
    string public text;

    constructor(string memory _text) {
        text = _text;
    }
}

// There are 2 ways to initialize parent contract with parameters.

// Pass the parameters here in the inheritance list.
contract B is X("Input to X"), Y("Input to Y") {

}

contract C is X, Y {
    // Pass the parameters here in the constructor,
    // similar to function modifiers.
    constructor(string memory _name, string memory _text) X(_name) Y(_text) {}
}

// Parent constructors are always called in the order of inheritance
// regardless of the order of parent contracts listed in the constructor of the child contract.

// Order of constructors called:
// 1. X
// 2. Y
// 3. D
contract D is X, Y {
    constructor() X("X was called") Y("Y was called") {}
}

// Order of constructors called:
// 1. X
// 2. Y
// 3. E
contract E is X, Y {
    constructor() Y("Y was called") X("X was called") {}
}
```
