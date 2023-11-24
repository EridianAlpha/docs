# Arrays

There are two types of arrays in Solidity:

* Fixed arrays
* Dynamic arrays

```solidity
// Array with a fixed length of 2 elements:
uint[2] fixedArray;

// another fixed Array, can contain 5 strings:
string[5] stringArray;

// a dynamic Array - has no fixed size, can keep growing:
uint[] dynamicArray;
```

You can also create an array of structs:

```
People[] public people; // dynamic Array, we can keep adding to it
```

Remember that state variables are stored permanently in the blockchain. So creating a dynamic array of structs like this can be useful for storing structured data in your contract, kind of like a database.

### Public Arrays

You can declare an array as public, and Solidity will automatically create a getter method for it. The syntax looks like:

```solidity
struct People {
  uint age;
  string name;
}

People[] public people;
```

### Working with Arrays

Create new `People` and add them to our `people` array.

```solidity
// Create a new Person:
People satoshi = People(172, "Satoshi");

// Add that person to the Array:
people.push(satoshi);
```

We can also combine these together and do them in one line of code to keep things clean:

```solidity
people.push(Person(16, "Vitalik"));
```

Note that `array.push()` adds something to the end of the array, so the elements are in the order we added them. See the following example:

```solidity
uint[] numbers;
numbers.push(5);
numbers.push(10);
numbers.push(15);
// numbers is now equal to [5, 10, 15]
```

### Example

* [https://solidity-by-example.org/array/](https://solidity-by-example.org/array/)

{% code fullWidth="false" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract Array {
    // Several ways to initialize an array
    uint[] public arr;
    uint[] public arr2 = [1, 2, 3];
    // Fixed sized array, all elements initialize to 0
    uint[10] public myFixedSizeArr;

    function get(uint i) public view returns (uint) {
        return arr[i];
    }

    // Solidity can return the entire array.
    // But this function should be avoided for arrays that 
    // can grow indefinitely in length.
    function getArr() public view returns (uint[] memory) {
        return arr;
    }

    function push(uint i) public {
        // Append to array
        // This will increase the array length by 1.
        arr.push(i);
    }

    function pop() public {
        // Remove last element from array
        // This will decrease the array length by 1
        arr.pop();
    }

    function getLength() public view returns (uint) {
        return arr.length;
    }

    function remove(uint index) public {
        // Delete does not change the array length.
        // It resets the value at index to it's default value,
        // in this case 0
        delete arr[index];
    }

    function examples() external {
        // create array in memory, only fixed size can be created
        uint[] memory a = new uint[](5);
    }
}
```
{% endcode %}

### Examples - Removing array element

* [https://solidity-by-example.org/array/](https://solidity-by-example.org/array/)

{% code fullWidth="false" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract ArrayReplaceFromEnd {
    uint[] public arr;

    // Deleting an element creates a gap in the array.
    // One trick to keep the array compact is to
    // move the last element into the place to delete.
    function remove(uint index) public {
        // Move the last element into the place to delete
        arr[index] = arr[arr.length - 1];
        // Remove the last element
        arr.pop();
    }

    function test() public {
        arr = [1, 2, 3, 4];

        remove(1);
        // [1, 4, 3]
        assert(arr.length == 3);
        assert(arr[0] == 1);
        assert(arr[1] == 4);
        assert(arr[2] == 3);

        remove(2);
        // [1, 4]
        assert(arr.length == 2);
        assert(arr[0] == 1);
        assert(arr[1] == 4);
    }
}
```
{% endcode %}
