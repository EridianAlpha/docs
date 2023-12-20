# Functions

## Function Types

<table><thead><tr><th width="179">Type</th><th>Description</th></tr></thead><tbody><tr><td><code>non-constant</code></td><td><ul><li>The default function type and doesn't get specified as a <a href="function-modifiers.md">modifier</a>.</li><li>The function can modify the state of the contract on the blockchain. Non-constant functions can write to the contract's storage, emit events, create other contracts, and use <code>selfdestruct</code>.</li></ul></td></tr><tr><td><code>view</code></td><td><ul><li>Doesn't cost gas when called directly (externally by a user).</li><li>Does cost gas when called by another function or contract.</li><li>Reads the state of the blockchain but can't modify it.</li></ul></td></tr><tr><td><code>pure</code></td><td><ul><li>Doesn't cost gas when called directly.</li><li>Does cost gas when called by another function or contract.</li><li>Does not read the state of the blockchain, only memory and calldata.</li></ul></td></tr></tbody></table>



## Code Conventions

* Function parameters and private function names start with an underscore `_`

```solidity
uint[] numbers;

function _addToArray(uint _number) private {
  numbers.push(_number);
}
```

## Visibility Specifiers

* Info
  * [https://docs.soliditylang.org/en/v0.8.13/cheatsheet.html#function-visibility-specifiers](https://docs.soliditylang.org/en/v0.8.13/cheatsheet.html#function-visibility-specifiers)
  * [https://medium.com/@yangnana11/solidity-function-types-4ad4e5de6d56](https://medium.com/@yangnana11/solidity-function-types-4ad4e5de6d56)
* Variables default to `internal` if no visibility specifier is given
* State variables can be declared as `public`, `private`, or `internal` but not `external`
  * [https://solidity-by-example.org/visibility/](https://solidity-by-example.org/visibility/)

<table data-full-width="true"><thead><tr><th width="141">Visibility</th><th>Description</th></tr></thead><tbody><tr><td><code>internal</code></td><td><ul><li>Default visibility specifier when none are specified.</li><li>Internal functions can only be called inside the current contract (more specifically, inside the current code unit, which also includes internal library functions and inherited functions) because they cannot be executed outside of the context of the current contract.</li><li>Calling an internal function is realized by jumping to its entry label, just like when calling a function of the current contract internally.</li><li>Those functions and state variables can only be accessed internally (i.e. from within the current contract or contracts deriving from it), without using <code>this</code>.</li></ul></td></tr><tr><td><code>external</code></td><td><ul><li>External functions consist of an address and a function signature and they can be passed via and returned from external function calls.</li><li>Can be called from outside, can’t be called from inside (functions from same contract, functions from inherited contracts).</li><li>External functions are part of the contract interface, which means they can be called from other contracts and via transactions.</li><li>An external function <code>f</code> cannot be called internally (i.e. <code>f()</code> does not work, but <code>this.f()</code> works).</li><li>External functions are sometimes more efficient when they receive large arrays of data.</li></ul></td></tr><tr><td><code>private</code></td><td><ul><li>Private functions can only be called from inside the current contract, even the inherited contracts can’t call them.</li><li>Private functions and state variables are only visible for the contract they are defined in and not in derived contracts.</li></ul></td></tr><tr><td><code>public</code></td><td><ul><li>Public functions can be called from anywhere.</li><li>Public functions are part of the contract interface and can be either called internally or via messages.</li><li>For public state variables, an automatic getter function is generated.</li></ul></td></tr></tbody></table>



### Function Declarations

A function declaration in Solidity looks like the following:

```solidity
function eatHamburgers(string memory _name, uint _amount) public {
}
```

This is a function named `eatHamburgers` that takes 2 parameters: a string and a uint. For now the body of the function is empty. Note that we're specifying the function visibility as public. We're also providing instructions about where the `_name` variable should be stored in memory. This is required for all reference types such as `arrays`, `structs`, `mappings`, and `strings`.

What is a reference type you ask? Well, there are two ways in which you can pass an argument to a Solidity function:

* By value, which means that the Solidity compiler creates a new copy of the parameter's value and passes it to your function. This allows your function to modify the value without worrying that the value of the initial parameter gets changed.
* By reference, which means that your function is called with a... reference to the original variable. Thus, if your function changes the value of the variable it receives, the value of the original variable gets changed.

{% hint style="info" %}
It's convention (but not required) to start function parameter variable names with an underscore (\_) in order to differentiate them from global variables.
{% endhint %}

You would call this function like so:

```
eatHamburgers("vitalik", 100);
```

### Private / Public functions

{% hint style="warning" %}
In Solidity, functions are `public` by default. This means anyone (or any other contract) can call your contract's function and execute its code.
{% endhint %}

Obviously, this isn't always desirable and can make your contract vulnerable to attacks. Thus it's good practice to make all functions `private`, and then only make `public` the functions you want to expose to the world.

Let's look at how to declare a private function:

```solidity
uint[] numbers;

function _addToArray(uint _number) private {
  numbers.push(_number);
}
```

This means only other functions within our contract will be able to call this function and add to the `numbers`array.

As you can see, we use the keyword `private`after the function name.

{% hint style="info" %}
It's convention to start private function names with an underscore `_`.
{% endhint %}

### Internal and External functions

In addition to `public` and `private`, Solidity has two more types of visibility for functions:

* `internal`
  * Similar `private`, except that it's also accessible to contracts that inherit from this contract.
* `external`
  * Similar to `public`, except that these functions can ONLY be called outside the contract — they can't be called by other functions inside that contract.
  * For declaring internal or external functions, the syntax is the same as private and public:

```solidity
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string memory) {
    baconSandwichesEaten++;
    // We can call this here because it's internal
    eat();
  }
}
```

## Handling Multiple Return Values

This `getKitty` function is the first example we've seen that returns multiple values. Let's look at how to handle them:

```solidity
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // This is how you do multiple assignment:
  (a, b, c) = multipleReturns();
}

// Or if we only cared about one of the values:
function getLastReturnValue() external {
  uint c;
  // We can just leave the other fields blank:
  (,,c) = multipleReturns();
}
```

## Example

* [https://solidity-by-example.org/function/](https://solidity-by-example.org/function/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract Function {
    // Functions can return multiple values.
    function returnMany()
        public
        pure
        returns (
            uint,
            bool,
            uint
        )
    {
        return (1, true, 2);
    }

    // Return values can be named.
    function named()
        public
        pure
        returns (
            uint x,
            bool b,
            uint y
        )
    {
        return (1, true, 2);
    }

    // Return values can be assigned to their name.
    // In this case the return statement can be omitted.
    function assigned()
        public
        pure
        returns (
            uint x,
            bool b,
            uint y
        )
    {
        x = 1;
        b = true;
        y = 2;
    }

    // Use destructuring assignment when calling another
    // function that returns multiple values.
    function destructuringAssignments()
        public
        pure
        returns (
            uint,
            bool,
            uint,
            uint,
            uint
        )
    {
        (uint i, bool b, uint j) = returnMany();

        // Values can be left out.
        (uint x, , uint y) = (4, 5, 6);

        return (i, b, j, x, y);
    }

    // Cannot use map for either input or output

    // Can use array for input
    function arrayInput(uint[] memory _arr) public {}

    // Can use array for output
    uint[] public arr;

    function arrayOutput() public view returns (uint[] memory) {
        return arr;
    }
}
```
