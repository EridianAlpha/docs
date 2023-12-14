# Data - Storage Layout

{% @github-files/github-code-block %}

* [https://docs.soliditylang.org/en/latest/internals/layout\_in\_storage.html](https://docs.soliditylang.org/en/latest/internals/layout\_in\_storage.html)&#x20;
* Each storage slot is `32 bytes` long and represents the bytes version of the object

{% hint style="info" %}
Constants and immutable variables are not in storage and don't take up a storage slot, as they are considered part of the bytecode of the contract.
{% endhint %}

{% code fullWidth="true" %}
```solidity
[0] 0x00...19       <--    uint256 favoriteNumber = 25;                 // Hex representation of 25
[1] 0x00...01       <--    bool someBool = true;                        // Hex value of 1 for true 
[2] 0x00...01       <--    uint256[] myArray;                           // Storage slot only contains array length
[3] 0x00...00       <--    mapping(address => uint256) public balances  // Empty storage slot to show it's a mapping
[4]
...

// ARRAYS
// Storage locations for data in array myArray from storage slot [2]
    // 2 is the slot number containing the length
    // i is the index of the array item (not used for push as it just gets added to the end of the array)
[keccak256(2)]      <--    myArray.push(222);
[keccak256(2) + i]  <--    myArray[i];

// MAPPINGS
// Storage locations for data in mapping balances from storage slot [3]
    // 3 is the slot where `balances` is stored
    // 0x123... is the mapping key (an address in this example)
[keccak256(abi.encode(0x123..., 3))]    <-- balances[0x123...]
```
{% endcode %}
