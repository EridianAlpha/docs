# Objects & Types

### Types of Objects

* [https://docs.soliditylang.org/en/v0.8.13/types.html](https://docs.soliditylang.org/en/v0.8.13/types.html)

<table data-full-width="true"><thead><tr><th width="388">Type</th><th>Description</th></tr></thead><tbody><tr><td><pre class="language-solidity"><code class="lang-solidity">uint
uint8
uint16
uint32
uint64
uint256
</code></pre></td><td><ul><li>Unsigned integer (whole number)</li><li><p>256 is the default if nothing is specified</p><ul><li>But it's good to be specific and use <code>uint256</code></li></ul></li><li>Initializes as default <code>0</code> if not assigned a value as that is the null value in Solidity</li><li>Smallest is <code>unit8</code> as 8 bits is a byte</li></ul></td></tr><tr><td><pre class="language-solidity"><code class="lang-solidity">int
</code></pre></td><td><ul><li>Positive or negative whole number</li></ul></td></tr><tr><td><pre class="language-solidity"><code class="lang-solidity">bytes
bytes2
bytes3
bytes5
bytes22
bytes32
</code></pre></td><td><ul><li><code>bytes32</code> is the max size allowed</li><li><p><code>bytes</code> can have "any size"?</p><ul><li>But I think that will still limit the actual content to 32 bytes</li></ul></li></ul></td></tr><tr><td><pre class="language-solidity"><code class="lang-solidity">string
</code></pre></td><td><ul><li>Actually a type of <code>bytes</code> in the background, but only used for text</li></ul></td></tr><tr><td><pre class="language-solidity"><code class="lang-solidity">bool
</code></pre></td><td><ul><li>boolean</li><li>true/false</li></ul></td></tr><tr><td><pre class="language-solidity"><code class="lang-solidity">address
</code></pre></td><td><ul><li>An address!</li></ul></td></tr></tbody></table>

```solidity
contract SimpleStorage {
    bool hasFavouriteNumber = true;
    uint256 favouriteNumber = 5;
    string favouriteNumberInText = "Five";
    int256 favouriteInt = -5;
    address myAddress = 0x5E666460E5BB4A8Bb14E805478176c36f3b293AB;
    bytes32 favouriteBytes = "cat";
}
```

### Example

* [https://solidity-by-example.org/primitives/](https://solidity-by-example.org/primitives/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract Primitives {
    bool public boo = true;

    /*
    uint stands for unsigned integer, meaning non negative integers
    different sizes are available
        uint8   ranges from 0 to 2 ** 8 - 1
        uint16  ranges from 0 to 2 ** 16 - 1
        ...
        uint256 ranges from 0 to 2 ** 256 - 1
    */
    uint8 public u8 = 1;
    uint public u256 = 456;
    uint public u = 123; // uint is an alias for uint256

    /*
    Negative numbers are allowed for int types.
    Like uint, different ranges are available from int8 to int256
    
    int256 ranges from -2 ** 255 to 2 ** 255 - 1
    int128 ranges from -2 ** 127 to 2 ** 127 - 1
    */
    int8 public i8 = -1;
    int public i256 = 456;
    int public i = -123; // int is same as int256

    // minimum and maximum of int
    int public minInt = type(int).min;
    int public maxInt = type(int).max;

    address public addr = 0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c;

    /*
    In Solidity, the data type byte represent a sequence of bytes. 
    Solidity presents two type of bytes types :

     - fixed-sized byte arrays
     - dynamically-sized byte arrays.
     
     The term bytes in Solidity represents a dynamic array of bytes. 
     It’s a shorthand for byte[] .
    */
    bytes1 a = 0xb5; //  [10110101]
    bytes1 b = 0x56; //  [01010110]

    // Default values
    // Unassigned variables have a default value
    bool public defaultBoo; // false
    uint public defaultUint; // 0
    int public defaultInt; // 0
    address public defaultAddr; // 0x0000000000000000000000000000000000000000
}
```







