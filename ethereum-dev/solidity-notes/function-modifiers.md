# Function Modifiers

## view / pure

When a function doesn't actually change state in Solidity — e.g. it doesn't change any values or write anything we could declare it as a `view`function, meaning it's only viewing the data but not modifying it:

```solidity
pragma solidity ^0.8.0;

contract GreetingContract {
    string private greeting;

    constructor() {
        greeting = "Hello, World!";
    }

    function sayHello() public view returns (string memory) {
        return greeting;
    }
}
```

Solidity also contains `pure`functions, which means you're not even accessing any data in the app. Consider the following:

```solidity
function _multiply(uint a, uint b) private pure returns (uint) {
    return a * b;
}
```

This function doesn't even read from the state of the app — its return value depends only on its function parameters. So in this case we would declare the function as `pure`.



## Ownable Contracts

Below is the `Ownable` contract taken from the `OpenZeppelin` Solidity library. OpenZeppelin is a library of secure and community-vetted smart contracts that you can use in your own DApps.

* [https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.1/contracts/access/Ownable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.1/contracts/access/Ownable.sol)

{% code fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
// OpenZeppelin Contracts (last updated v5.0.0) (access/Ownable.sol)

pragma solidity ^0.8.20;

import {Context} from "../utils/Context.sol";

/**
 * @dev Contract module which provides a basic access control mechanism, where
 * there is an account (an owner) that can be granted exclusive access to
 * specific functions.
 *
 * The initial owner is set to the address provided by the deployer. This can
 * later be changed with {transferOwnership}.
 *
 * This module is used through inheritance. It will make available the modifier
 * `onlyOwner`, which can be applied to your functions to restrict their use to
 * the owner.
 */
abstract contract Ownable is Context {
    address private _owner;

    /**
     * @dev The caller account is not authorized to perform an operation.
     */
    error OwnableUnauthorizedAccount(address account);

    /**
     * @dev The owner is not a valid owner account. (eg. `address(0)`)
     */
    error OwnableInvalidOwner(address owner);

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    /**
     * @dev Initializes the contract setting the address provided by the deployer as the initial owner.
     */
    constructor(address initialOwner) {
        if (initialOwner == address(0)) {
            revert OwnableInvalidOwner(address(0));
        }
        _transferOwnership(initialOwner);
    }

    /**
     * @dev Throws if called by any account other than the owner.
     */
    modifier onlyOwner() {
        _checkOwner();
        _;
    }

    /**
     * @dev Returns the address of the current owner.
     */
    function owner() public view virtual returns (address) {
        return _owner;
    }

    /**
     * @dev Throws if the sender is not the owner.
     */
    function _checkOwner() internal view virtual {
        if (owner() != _msgSender()) {
            revert OwnableUnauthorizedAccount(_msgSender());
        }
    }

    /**
     * @dev Leaves the contract without owner. It will not be possible to call
     * `onlyOwner` functions. Can only be called by the current owner.
     *
     * NOTE: Renouncing ownership will leave the contract without an owner,
     * thereby disabling any functionality that is only available to the owner.
     */
    function renounceOwnership() public virtual onlyOwner {
        _transferOwnership(address(0));
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Can only be called by the current owner.
     */
    function transferOwnership(address newOwner) public virtual onlyOwner {
        if (newOwner == address(0)) {
            revert OwnableInvalidOwner(address(0));
        }
        _transferOwnership(newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Internal function without access restriction.
     */
    function _transferOwnership(address newOwner) internal virtual {
        address oldOwner = _owner;
        _owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
}
```
{% endcode %}

The `Ownable`contract does the following:

1. When a contract is created, its constructor sets the `Ownable` to `msg.sender`(the person who deployed it).
2. It adds an `onlyOwner` modifier, which can restrict access to certain functions to only the `Ownable`.
3. It allows you to transfer the contract to a new `Ownable`.

`onlyOwner` is such a common requirement for contracts that most Solidity DApps start with a copy/paste or import of this `Ownable` contract, and then their first contract inherits from it.

## Function Modifiers - Generic

A function modifier looks just like a function, but uses the keyword `modifier` instead of the keyword `function`. And it can't be called directly like a function can — instead we can attach the modifier's name at the end of a function definition to change that function's behavior.

Let's take a closer look by examining `onlyOwner` (see code extract above)

* Notice the `onlyOwner` modifier on the `renounceOwnership` function.
* When you call `renounceOwnership`, the code inside `onlyOwner` executes first.&#x20;
* Then when it hits the `_;`statement in `onlyOwner`, it goes back and executes the code inside renounceOwnership.
* While there are other ways you can use modifiers, one of the most common use-cases is to add a quick require check before a function executes.
* In the case of onlyOwner, adding this modifier to a function makes it so only the owner of the contract (you, if you deployed it) can call that function.

{% hint style="info" %}
Giving the owner special powers over the contract like this is often necessary, but it could also be used maliciously. For example, the owner could add a backdoor function.
{% endhint %}

* Modifiers can all be stacked together on a function definition.
* Modifiers are executed in the order they are listed in the function declaration.

```solidity
function test() external view onlyOwner anotherModifier { /* ... */ }
```

1. First, the logic inside the `onlyOwner` modifier is executed. This modifier typically checks whether the caller of the function is the owner of the contract.
2. After `onlyOwner` completes its execution, `anotherModifier` is executed next. The specific logic of this modifier depends on its implementation.
3. Finally, if all modifiers execute successfully (i.e., none of them revert), the body of the `test` function is executed.

## Function Modifiers with Arguments

Function modifiers can also take arguments.

```solidity
// A mapping to store a user's age:
mapping (uint => uint) public age;

// Modifier that requires this user to be older than a certain age:
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// Must be older than 16 to drive a car (in the US, at least).
// We can call the `olderThan` modifier with arguments like so:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // Some function logic
}
```

* You can see here that the `olderThan` modifier takes arguments just like a function does.
* And that the `driveCar` function passes its arguments to the modifier.

## Payable Modifier

`payable`functions are a special type of function that can receive ETH.

* This allows for some really interesting logic, like requiring a certain payment to the contract in order to execute a function.

```solidity
contract OnlineStore {
  function buySomething() external payable {
    // Check to make sure 0.001 ether was sent to the function call:
    require(msg.value == 0.001 ether);
    // If so, some logic to transfer the digital item to the caller of the function:
    transferThing(msg.sender);
  }
}
```

* Here, `msg.value`is a way to see how much ETH was sent to the contract, and ether is a built-in unit.
* What happens here is that someone would call the function from web3.js (from the DApp's JavaScript front-end) as follows:

{% code overflow="wrap" %}
```javascript
// Assuming `OnlineStore` points to your contract on Ethereum:
OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})
```
{% endcode %}

* Notice the `value`field, where the javascript function call specifies how much ether to send (0.001).
* If you think of the transaction like an envelope, and the parameters you send to the function call are the contents of the letter you put inside, then adding a `value` is like putting cash inside the envelope — the letter and the money get delivered together to the recipient.

{% hint style="warning" %}
If a function is not marked payable and you try to send Ether to it as above, the function will reject your transaction.
{% endhint %}

### Function Modifier Example

* [https://solidity-by-example.org/function-modifier](https://solidity-by-example.org/function-modifier)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract FunctionModifier {
    // We will use these variables to demonstrate how to use modifiers.
    address public owner;
    uint public x = 10;
    bool public locked;

    constructor() {
        // Set the transaction sender as the owner of the contract.
        owner = msg.sender;
    }

    // Modifier to check that the caller is the owner of the contract.
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        // Underscore is a special character only used inside a function modifier
        // and it tells Solidity to execute the rest of the code.
        _;
    }

    // Modifiers can take inputs. This modifier checks that the
    // address passed in is not the zero address.
    modifier validAddress(address _addr) {
        require(_addr != address(0), "Not valid address");
        _;
    }

    function changeOwner(address _newOwner) public onlyOwner validAddress(_newOwner) {
        owner = _newOwner;
    }

    // Modifiers can be called before and / or after a function.
    // This modifier prevents a function from being called while it is still executing.
    modifier noReentrancy() {
        require(!locked, "No reentrancy");

        locked = true;
        _;
        locked = false;
    }

    function decrement(uint i) public noReentrancy {
        x -= i;
        if (i > 1) {
            decrement(i - 1);
        }
    }
}
```
