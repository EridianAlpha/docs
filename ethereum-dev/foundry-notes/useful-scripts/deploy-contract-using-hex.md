# Deploy Contract Using Hex

* Foundry script file

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";

contract Exploit is Script {
    function deployContractFromHex() public returns (address) {
        bytes memory bytecode = "0x608060...0033";

        vm.startBroadcast();
        (bool success, ) = address(0).call{value: 0, gas: 5_000_000}(bytecode);
        require(success, "Contract deployment failed");
        vm.stopBroadcast();

        // Calculate the address of the deployed contract
        address deployer = address(this);
        uint256 nonce = vm.getNonce(deployer) - 1; // Nonce at the time of deployment
        address deployedAddress = address(
            uint160(
                uint256(
                    keccak256(
                        abi.encodePacked(
                            bytes1(0xd6),
                            bytes1(0x94),
                            deployer,
                            nonce
                        )
                    )
                )
            )
        );

        console.log("Deployed Contract Address:", deployedAddress);
        return deployedAddress;
    }
}
```
