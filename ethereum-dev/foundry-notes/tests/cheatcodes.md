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

## prank

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

## startPrank

Sets `msg.sender` **for all subsequent calls** until [`stopPrank`](https://book.getfoundry.sh/cheatcodes/stop-prank.html) is called.

If the alternative signature of `startPrank` is used, then `tx.origin` is set as well for all subsequent calls.

{% code title="startPrank example" %}
```solidity
function startPrank(address) external;
function startPrank(address sender, address origin) external;
```
{% endcode %}

## stopPrank

Stops an active prank started by [`startPrank`](https://book.getfoundry.sh/cheatcodes/start-prank.html), resetting `msg.sender` and `tx.origin` to the values before `startPrank` was called.

{% code title="startPrank example" %}
```solidity
function stopPrank() external;
```
{% endcode %}

## txGasPrice

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

## warp

Sets `block.timestamp`

```solidity
vm.warp(1641070800);
emit log_uint(block.timestamp); // 1641070800
```

## roll

Sets `block.number`.

```solidity
vm.roll(100);
emit log_uint(block.number); // 100
```

## recordLogs

Tells the VM to start recording all the emitted events. To access them, use `getRecordedLogs`.

{% code fullWidth="false" %}
```solidity
vm.recordLogs();                                // Start recording logs
raffle.performUpkeep("");                       // Call function that emits event log
Vm.Log[] memory entries = vm.getRecordedLogs(); // Store emitted log

// Access stored log by knowing exactly which logs are emitted
// .topics[0] would refer to the entire event
// .topics[1] is for the first actual topic
// All logs are recorded as bytes32 in foundry
bytes32 requestId = entries[1].topics[1];       
```
{% endcode %}

## expectEmit

Assert a specific log is emitted during the next call.

```solidity
function expectEmit() external;
function expectEmit(
    bool checkTopic1,
    bool checkTopic2,
    bool checkTopic3,
    bool checkData
) external;

function expectEmit(address emitter) external;
function expectEmit(
    bool checkTopic1,
    bool checkTopic2,
    bool checkTopic3,
    bool checkData,
    address emitter
) external;
```

1. Call the cheat code, specifying whether we should check the first, second or third topic, and the log data (`expectEmit``()` checks them all). Topic 0 is always checked.
2. Emit the event we are supposed to see during the next call.
3. Perform the call.

You can perform steps 1 and 2 multiple times to match a _sequence_ of events in the next call.

If the event is not available in the current scope (e.g. if we are using an interface, or an external smart contract), we can define the event ourselves with an identical event signature.

There are 2 varieties of `expectEmit`:

* **Without checking the emitter address**: Asserts the topics match **without** checking the emitting address.
* **With `address`**: Asserts the topics match and that the emitting address matches.

{% hint style="info" %}
**Matching sequences**

In functions that emit a lot of events, it's possible to "skip" events and only match a specific sequence, but this sequence must always be in order. As an example, let's say a function emits events: `A, B, C, D, E, F, F, G`.

`expectEmit` will be able to match ranges with and without skipping events in between:

* `[A, B, C]` is valid.
* `[B, D, F]` is valid.
* `[G]` or any other single event combination is valid.
* `[B, A]` or similar out-of-order combinations are **invalid** (events must be in order).
* `[C, F, F]` is valid.
* `[F, F, C]` is **invalid** (out of order).
{% endhint %}

### expectEmit Examples

{% code title="This does not check the emitting address." fullWidth="true" %}
```solidity
event Transfer(address indexed from, address indexed to, uint256 amount);

function testERC20EmitsTransfer() public {
    vm.expectEmit();

    // We emit the event we expect to see.
    emit MyToken.Transfer(address(this), address(1), 10);

    // We perform the call.
    myToken.transfer(address(1), 10);
}
```
{% endcode %}

{% code title="This does check the emitting address." fullWidth="true" %}
```solidity
event Transfer(address indexed from, address indexed to, uint256 amount);

function testERC20EmitsTransfer() public {
    // We check that the token is the event emitter by passing the address.
    vm.expectEmit(address(myToken));
    emit MyToken.Transfer(address(this), address(1), 10);

    // We perform the call.
    myToken.transfer(address(1), 10);
}
```
{% endcode %}

<pre class="language-solidity" data-title="We can also assert that multiple events are emitted in a single call." data-full-width="true"><code class="lang-solidity"><strong>function testERC20EmitsBatchTransfer() public {
</strong>    // We declare multiple expected transfer events
    for (uint256 i = 0; i &#x3C; users.length; i++) {
        // Here we use the longer signature for demonstration purposes. This call checks
        // topic0 (always checked), topic1 (true), topic2 (true), NOT topic3 (false), and data (true).
        vm.expectEmit(true, true, false, true);
        emit Transfer(address(this), users[i], 10);
    }

    // We also expect a custom `BatchTransfer(uint256 numberOfTransfers)` event.
    vm.expectEmit(false, false, false, true);
    emit BatchTransfer(users.length);

    // We perform the call.
    myToken.batchTransfer(users, 10);
}
</code></pre>

{% code title="This example fails, as the expected event is not emitted on the next call." fullWidth="true" %}
```solidity
event Transfer(address indexed from, address indexed to, uint256 amount);

function testERC20EmitsTransfer() public {
    // We check that the token is the event emitter by passing the address as the fifth argument.
    vm.expectEmit(true, true, false, true, address(myToken));
    emit MyToken.Transfer(address(this), address(1), 10);

    // We perform an unrelated call that won't emit the intended event,
    // making the cheatcode fail.
    myToken.approve(address(this), 1e18);
    // We perform the call, but it will have no effect as the cheatcode has already failed.
    myToken.transfer(address(1), 10);
}
```
{% endcode %}
