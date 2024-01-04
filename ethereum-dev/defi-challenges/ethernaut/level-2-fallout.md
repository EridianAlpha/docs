# Level 2 - Fallout ⏺

{% embed url="https://ethernaut.openzeppelin.com/level/2" %}

### Level Setup

{% hint style="info" %}
Claim ownership of the contract below to complete this level.

&#x20; Things that might help

* Solidity Remix IDE
{% endhint %}

### Level Contract

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;


  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```
{% endcode %}

### Exploit

The&#x20;

1. Start by

{% code title="" %}
```javascript
```
{% endcode %}





2. Submit instance... 🥳

### Completion Message

{% hint style="info" %}

{% endhint %}

### Notes

*
