---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Cheatcodes

To manipulate the state of the blockchain, as well as test for specific reverts and events, Foundry is shipped with a set of cheatcodes.

Cheatcodes are accessed using the `vm` instance available in Forge Standard Library's `Test` contract.

### prank

Sets `msg.sender` to the specified address **for the next call**. "The next call" includes static calls as well, but not calls to the cheat code address.

If the alternative signature of `prank` is used, then `tx.origin` is set as well for the next call.

{% code title="prank example" %}
```solidity
/// function withdraw() public {
///     require(msg.sender == owner);

vm.prank(owner);
myContract.withdraw(); // [PASS]
```
{% endcode %}

### startPrank

Sets `msg.sender` **for all subsequent calls** until [`stopPrank`](https://book.getfoundry.sh/cheatcodes/stop-prank.html) is called.

If the alternative signature of `startPrank` is used, then `tx.origin` is set as well for all subsequent calls.

{% code title="startPrank example" %}
```solidity
function startPrank(address) external;
function startPrank(address sender, address origin) external;
```
{% endcode %}

### stopPrank

Stops an active prank started by [`startPrank`](https://book.getfoundry.sh/cheatcodes/start-prank.html), resetting `msg.sender` and `tx.origin` to the values before `startPrank` was called.

{% code title="startPrank example" %}
```solidity
function stopPrank() external;
```
{% endcode %}

### txGasPrice

Sets `tx.gasprice` for the rest of the transaction.

{% code title="txGasPrice example" %}
```solidity
function txGasPrice(uint256) external;
function txGasPrice(uint256 newGasPrice) external;

// Example usage
uint256 gasStart = gasleft();
vm.txGasPrice(GAS_PRICE);
// ...Do things...
uint256 gasEnd = gasleft();
uint256 gasUsed = (gasStart - gasEnd) * GAS_PRICE;
console.log("Gas used: ", gasUsed);
```
{% endcode %}

