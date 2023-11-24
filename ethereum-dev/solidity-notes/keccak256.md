# Keccak256

* [https://medium.com/0xcode/hashing-functions-in-solidity-using-keccak256-70779ea55bb0](https://medium.com/0xcode/hashing-functions-in-solidity-using-keccak256-70779ea55bb0)

Ethereum has the hash function `keccak256` built-in, which is a version of SHA3. A hash function basically maps an input into a random 256-bit hexadecimal number. A slight change in the input will cause a large change in the hash.

`keccak256` expects a single parameter of type bytes. This means that we have to "pack" any parameters before calling keccak256:

```solidity
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256(abi.encodePacked("aaaab"));

//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
keccak256(abi.encodePacked("aaaac"));
```

As you can see, the returned values are totally different despite only a 1-character change in the input.
