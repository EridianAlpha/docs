# Data - Storage vs Memory

{% @github-files/github-code-block %}

### Data Locations

The EVM can access and store information in six places:

1. Stack
2. Memory
   * Temporary variables that can be modified
3. Storage
   * Permanent variables that can be modified
4. Calldata
   * Temporary variables that can't be modified
5. Code
6. Logs

Data locations can only be specified for types:

* Array
  * Strings
* Struct
* Mapping

{% hint style="info" %}
**A location specifier is not needed for `unit` as Solidity knows that it will be `memory`**
{% endhint %}

### Calldata

* The variable only exists temporarily and can't be modified
* Similar to memory, but use `calldata` instead of `memory` if you don't plan on changing the variable
* `calldata` variable values can't be reassigned

### Storage vs Memory

In Solidity, there are two locations where you can store variables — in `storage`and in `memory`.

* `storage` refers to variables stored permanently on the blockchain.
* `memory` variables are temporary, and are erased between external function calls to your contract. Think of it like your computer's hard disk vs RAM.

Usually, these keywords aren't needed because Solidity handles them by default. State variables (variables declared outside of functions) are by default `storage` and written permanently to the blockchain, while variables declared inside functions are `memory` and will disappear when the function call ends.

However, there are times when you do need to use these keywords, namely when dealing with structs and arrays within functions:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ Seems pretty straightforward, but solidity will give you a warning
    // telling you that you should explicitly declare `storage` or `memory` here.

    // So instead, you should declare with the `storage` keyword, like:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...in which case `mySandwich` is a pointer to `sandwiches[_index]`
    // in storage, and...
    mySandwich.status = "Eaten!";
    // ...this will permanently change `sandwiches[_index]` on the blockchain.

    // If you just want a copy, you can use `memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...in which case `anotherSandwich` will simply be a copy of the 
    // data in memory, and...
    anotherSandwich.status = "Eaten!";
    // ...will just modify the temporary variable and have no effect 
    // on `sandwiches[_index + 1]`. But you can do this:
    sandwiches[_index + 1] = anotherSandwich;
    // ...if you want to copy the changes back into blockchain storage.
  }
}
```

* The Solidity compiler will give warnings to let you know when you should be using one of these keywords.
* Understand that there are cases where you'll need to explicitly declare `storage` or `memory`.

### Storage is Expensive

* One of the more expensive operations in Solidity is using `storage` — particularly writes.
* This is because every time you write or change a piece of data, it’s written permanently to the blockchain. Forever! Thousands of nodes across the world need to store that data on their hard drives, and this amount of data keeps growing over time as the blockchain grows. So there's a cost to doing that.
* In order to keep costs down, you want to avoid writing data to storage except when absolutely necessary. Sometimes this involves seemingly inefficient programming logic — like rebuilding an array in `memory`every time a function is called instead of simply saving that array in a variable for quick lookups.
* In most programming languages, looping over large data sets is expensive. But in Solidity, this is way cheaper than using `storage` if it's in an `external view`function, since `view` functions don't cost your users any gas. (And gas costs your users real money!).

### Declaring arrays in memory

* You can use the `memory` keyword with arrays to create a new array inside a function without needing to write anything to storage. The array will only exist until the end of the function call, and this is a lot cheaper gas-wise than updating an array in `storage` — free if it's a `view` function called externally.
* Here's how to declare an array in memory:

```solidity
function getArray() external pure returns(uint[] memory) {
  // Instantiate a new array in memory with a length of 3
  uint[] memory values = new uint[](3);

  // Put some values to it
  values[0] = 1;
  values[1] = 2;
  values[2] = 3;

  return values;
}
```

{% hint style="info" %}
`memory` arrays must be created with a length argument (in this example, 3). They currently cannot be resized like `storage` arrays can with `array.push()`, although this may be changed in a future version of Solidity.
{% endhint %}

### Example

* [https://solidity-by-example.org/data-locations/](https://solidity-by-example.org/data-locations/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract DataLocations {
    uint[] public arr;
    mapping(uint => address) map;
    struct MyStruct {
        uint foo;
    }
    mapping(uint => MyStruct) myStructs;

    function f() public {
        // call _f with state variables
        _f(arr, map, myStructs[1]);

        // get a struct from a mapping
        MyStruct storage myStruct = myStructs[1];
        // create a struct in memory
        MyStruct memory myMemStruct = MyStruct(0);
    }

    function _f(
        uint[] storage _arr,
        mapping(uint => address) storage _map,
        MyStruct storage _myStruct
    ) internal {
        // do something with storage variables
    }

    // You can return memory variables
    function g(uint[] memory _arr) public returns (uint[] memory) {
        // do something with memory array
    }

    function h(uint[] calldata _arr) external {
        // do something with calldata array
    }
}
```
