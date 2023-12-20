# Structs

* Structs can be declared outside of a contract and imported in another contract.
* Sometimes you need a more complex data type.
* For this, Solidity provides structs, allowing you to create more complicated data types that have multiple properties.

```solidity
struct People {
  uint256 favoriteNumber;
  string name;
}

// create a New Person:
People public person = People({favoriteNumber: 7, name: "Eridian"});
```

### Declaring and Importing Struct

{% code title="File that the struct is declared in" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;
// This is saved as 'StructDeclaration.sol'

struct Todo {
    string text;
    bool completed;
}
```
{% endcode %}

{% code title="File that imports the struct above" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import "./StructDeclaration.sol";

contract Todos {
    // An array of 'Todo' structs
    Todo[] public todos;
}
```
{% endcode %}

### Passing structs as arguments

* You can pass a storage pointer to a struct as an argument to a `private` or `internal` function.
* This is useful, for example, for passing around our `Zombie` structs between functions.

```solidity
function _doStuff(People storage _person) internal {
  // do stuff with _person
}
```

* This way we can pass a reference to our person into a function instead of passing in a person ID and looking it up.

### Struct packing to save gas

* There are other types of `uint`s: `uint8`, `uint16`, `uint32`, etc.
* Normally there's no benefit to using these sub-types because Solidity reserves 256 bits of storage regardless of the `uint` size.&#x20;
  * For example, using `uint8` instead of `uint (uint256)` won't save you any gas.
* But there's an exception to this: inside `struct`s.
* If you have multiple `uint`s inside a struct, using a smaller-sized `uint`when possible will allow Solidity to pack these variables together to take up less storage.&#x20;

```solidity
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` will cost less gas than `normal` because of struct packing
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30);
```

* For this reason, inside a struct you'll want to use the smallest integer sub-types you can get away with.
* You'll also want to cluster identical data types together (i.e. put them next to each other in the struct) so that Solidity can minimize the required storage space.
* For example, a struct with fields `uint c; uint32 a; uint32 b;` will cost less gas than a struct with fields `uint32 a; uint c; uint32 b;` because the `uint32` fields are clustered together.

### Example

* [https://solidity-by-example.org/structs/](https://solidity-by-example.org/structs/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract Todos {
    struct Todo {
        string text;
        bool completed;
    }

    // An array of 'Todo' structs
    Todo[] public todos;

    function create(string calldata _text) public {
        // 3 ways to initialize a struct
        // - calling it like a function
        todos.push(Todo(_text, false));

        // key value mapping
        todos.push(Todo({text: _text, completed: false}));

        // initialize an empty struct and then update it
        Todo memory todo;
        todo.text = _text;
        // todo.completed initialized to false

        todos.push(todo);
    }

    // Solidity automatically created a getter for 'todos' so you don't actually need this function.
    function get(uint _index) public view returns (string memory text, bool completed) {
        Todo storage todo = todos[_index];
        return (todo.text, todo.completed);
    }

    // update text
    function updateText(uint _index, string calldata _text) public {
        Todo storage todo = todos[_index];
        todo.text = _text;
    }

    // update completed
    function toggleCompleted(uint _index) public {
        Todo storage todo = todos[_index];
        todo.completed = !todo.completed;
    }
}
```







