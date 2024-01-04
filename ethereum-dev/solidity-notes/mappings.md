# Mappings

* A mapping is essentially a key-value store for storing and looking up data
* Mappings can't be in memory

```solidity
mapping(keyType => valueType)
```

* Here the key is an `address` and the value is a `uint256`
  * E.g. for a financial app, storing a uint that holds the user's account balance:

```solidity
mapping (address => uint) public addressToAccountBalance;
```

* Here the key is a uint and the value a string
  * e.g. Store / lookup usernames based on userId

```solidity
mapping (uint => string) userIdToName;
```

### Example

* [https://solidity-by-example.org/mapping/](https://solidity-by-example.org/mapping/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract Mapping {
    // Mapping from address to uint
    mapping(address => uint) public myMap;

    function get(address _addr) public view returns (uint) {
        // Mapping always returns a value.
        // If the value was never set, it will return the default value (0).
        return myMap[_addr];
    }

    function set(address _addr, uint _i) public {
        // Update the value at this address
        myMap[_addr] = _i;
    }

    function remove(address _addr) public {
        // Reset the value to the default value.
        delete myMap[_addr];
    }
}
```



### Example - Nested Mapping

* [https://solidity-by-example.org/mapping/](https://solidity-by-example.org/mapping/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract NestedMapping {
    // Nested mapping (mapping from address to another mapping)
    mapping(address => mapping(uint => bool)) public nested;

    function get(address _addr1, uint _i) public view returns (bool) {
        // You can get values from a nested mapping
        // even when it is not initialized
        return nested[_addr1][_i];
    }

    function set(
        address _addr1,
        uint _i,
        bool _boo
    ) public {
        nested[_addr1][_i] = _boo;
    }

    function remove(address _addr1, uint _i) public {
        delete nested[_addr1][_i];
    }
}
```
