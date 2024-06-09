# Data - Storage Layout

In Ethereum, all values in storage, whether they represent numbers, addresses, or array lengths, are stored as 32-byte hexadecimal values.

## **256-bit Address Space**

* In Ethereum, each smart contract has a storage space that can be thought of as a large array of 32-byte (256-bit) slots.
* This means there are `2^256` possible storage slots, each capable of holding 32 bytes of data.

## **Storage Slots**

* Each storage slot is 32 bytes (256 bits).
* Simple variables (e.g., uint256, address) typically occupy a single slot.
  * A `uint256` is 32 bytes so perfectly fills an entire slot.
  * An `address` is 20 bytes but still takes up a whole storage slot.
    * The remaining 12 bytes (96 bits) in the slot are padded with zeros.
  * A `bool` only requires 1 bit, it still occupies a full slot of 32 bytes, with 31 bytes padded with zeros.
* Complex data structures (e.g., mappings, arrays) utilize multiple slots, often determined by hash functions.

## Simple Variables

* Simple variables like `uint256`, `address`, `bool` occupy one storage slot.
* The slot index is determined by the order of declaration in the contract.

## Mappings

* Hash-Based Slot Calculation.
* Mappings are stored using a hash of the key and the slot number as the storage location.
* For a mapping `mapping(uint256 => uint256)`, the storage slot for a key `k` is `keccak256(abi.encodePacked(k, p))`, where `p` is the slot number of the mapping itself.
* E.g for a mapping 5 => 99 at mapping slot 3 the storage slot of the value would be
  * `keccak256(abi.encodePacked(5, 3))` is the slot storing 99
* A mapping doesn't have an empty storage slot to show it's a mapping, it has an empty storage slot because it needs the slot number to be able to calculate where it stored the values, and there's nothing to actually store in the slot.

## Arrays

* Static arrays have a base slot, and elements are stored in consecutive slots starting from that base slot. Each element occupies a full 32-byte storage slot, regardless of its actual size.
* Dynamic arrays store their length in the base slot, and elements are stored starting from `keccak256(baseSlot)`.

{% code fullWidth="true" %}
```solidity
uint256[3] public fixedArray; // Base slot, e.g., slot 0, 1, 2
uint256[] public dynamicArray; // Length at slot 1, elements at keccak256(1), keccak256(1)+1, ...
```
{% endcode %}

## Optimizing Storage with Packing

Solidity can pack multiple small variables into a single slot if they fit within the 32-byte limit and are declared consecutively. This is known as "storage packing."

```solidity
contract PackedStorage {
    uint128 public myUint128;       // Uses the first 16 bytes of slot 0
    uint128 public mySecondUint128; // Uses the next 16 bytes of slot 0
    uint64 public myUint64;         // Uses the first 8 bytes of slot 1
    uint64 public mySecondUint64;   // Uses the next 8 bytes of slot 1
    uint32 public myUint32;         // Uses the next 4 bytes of slot 1
    uint32 public mySecondUint32;   // Uses the next 4 bytes of slot 1
    uint32 public myThirdUint32;    // Uses the next 4 bytes of slot 1
    uint32 public myFourthUint32;   // Uses the next 4 bytes of slot 1
}
```

Storage packing can lead to significant gas savings by minimizing the number of storage slots used. However, care must be taken with the order of declaration to achieve optimal packing.

When you access a packed variable, Solidity automatically handles the correct segment of the storage slot. For example, if you access `myUint64`, Solidity retrieves the bytes 0-7 from slot 1 and interprets them as a `uint64`.

## Empty Slots and Interpretation

* Unused storage slots default to zero.
* There is no inherent indicator within the slot itself to distinguish whether it is an unused slot or a slot belonging to a mapping that has not been assigned a value yet.
* It is the responsibility of the compiler and the ABI to interpret storage correctly.
* The contract's bytecode and ABI contain the necessary information to differentiate between different types of storage (e.g., simple variables, arrays, mappings).

## Storage Example

{% @github-files/github-code-block %}

* [https://docs.soliditylang.org/en/latest/internals/layout\_in\_storage.html](https://docs.soliditylang.org/en/latest/internals/layout\_in\_storage.html)&#x20;
* Each storage slot is `32 bytes` long and represents the bytes version of the object

{% hint style="info" %}
Constants and immutable variables are not in storage and don't take up a storage slot, as they are considered part of the bytecode of the contract.
{% endhint %}

{% code fullWidth="true" %}
```solidity
[0] 0x00...19       <--    uint256 favoriteNumber = 25;                 // Hex representation of 25 (0x19)
[1] 0x00...01       <--    bool someBool = true;                        // Hex value of 1 for true (0x1)
[2] 0x00...01       <--    uint256[] myArray;                           // Storage slot only contains array length in hex
[3] 0x00...00       <--    mapping(address => uint256) public balances  // Empty storage slot since it's a mapping
...

// DYNAMIC ARRAYS
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
