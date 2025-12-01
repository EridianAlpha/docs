# Abstract Contracts

* [https://docs.soliditylang.org/en/v0.8.16/contracts.html#abstract-contracts](https://docs.soliditylang.org/en/v0.8.16/contracts.html#abstract-contracts)
* Contracts must be marked as abstract when at least one of their functions is not implemented or when they do not provide arguments for all of their base contract constructors
* Even if this is not the case, a contract may still be marked abstract, such as when you do not intend for the contract to be created directly
* Abstract contracts are similar to Interfaces but an interface is more limited in what it can declare.<br>
* An abstract contract is declared using the abstract keyword as shown in the following example
* Note that this contract needs to be defined as abstract, because the function utterance() is declared, but no implementation was provided (no implementation body { } was given)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

abstract contract Fruit {
    function color() public pure virtual returns (bytes32);
}

contract Apple is Fruit {
    function color() public pure override returns (bytes32) { return "green"; }
}
```
