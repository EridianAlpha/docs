# 🎛️ Technical Basics

## Why 0x?

In Ethereum, and more broadly in the world of cryptography and blockchain technology, the prefix `0x` is used everywhere... but why?

The prefix `0x` indicates that the following string is in hexadecimal (base 16) format. Hexadecimal is a numeral system that uses 16 symbols: 0-9 to represent values 0 to 9 and A-F to represent values 10 to 15.

```
Base 10:     0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
Hexadecimal: 0 1 2 3 4 5 6 7 8 9 A  B  C  D  E  F
```

By starting with `0x` Ethereum ensures no ambiguity about the data format.

{% hint style="info" %}
`0x` means "The string that follows is a hexadecimal!"
{% endhint %}

For example, an Ethereum address looks like `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266` and is represented in a hexadecimal format.

<table><thead><tr><th width="172">Type</th><th>Value</th></tr></thead><tbody><tr><td>Hexadecimal</td><td><code>f39Fd6e51aad88F6F4ce6aB8827279cffFb92266</code></td></tr><tr><td>Base 10</td><td><code>1390849295786071768276380950238675083608645509734</code></td></tr><tr><td>Base 2 (Binary)</td><td><code>1111001110011111110101101110010100011010101011011000100011110110111101001100111001101010101110001000001001110010011110011100111111111111101110010010001001100110</code></td></tr></tbody></table>

You'll notice from the table that the hexadecimal representation is the shortest number of characters.&#x20;

> Each hexadecimal digit represents four binary digits (bits).

This means a long binary string can be represented by a much shorter hexadecimal string, making it easier to read, write, and communicate.

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p><a href="https://medium.com/portis/part-two-turning-random-numbers-into-an-ethereum-address-3928f56b225c">https://medium.com/portis/part-two-turning-random-numbers-into-an-ethereum-address-3928f56b225c</a></p></figcaption></figure></div>

## uint8 - Smallest Solidity Example

A `uint8` in Solidity is an unsigned integer that can hold values from 0 to 255.&#x20;

It uses 8 bits (1 byte) to represent these values.

An 8-bit binary number can range from `00000000` (0) to `11111111` (255). For example, the number 150 in binary is `10010110`.

Group the 8 bits into two 4-bit groups (nibbles): `1001` and `0110`.

**Convert to Hexadecimal**:

* `1001` in binary is `9` in hexadecimal.
* `0110` in binary is `6` in hexadecimal.

Thus, the binary `10010110` becomes `0x96` in hexadecimal.

In Solidity, these are equivalent statements:

```solidity
uint8 public myNumberDecimal = 150;
uint8 public myNumberHex = 0x96;
```

## uint256 - Biggest Solidity Example

A `uint256` variable can hold values from `0` to `2^256 - 1`.

**Maximum `uint256` value**

* Base 10: `115792089237316195423570985008687907853269984665640564039457584007913129639935`
* Hexadecimal: `0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff`

A 256-bit binary number can range from 00000000…00000000 (0) to 11111111…11111111 (2^256 - 1).

* `uint256` means it is an unsigned integer that uses 256 bits.
* 256 bits = 32 bytes (since 1 byte = 8 bits).
