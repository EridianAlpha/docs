# Msg.sender

In Solidity, there are certain global variables that are available to all functions. One of these is `msg.sender`, which refers to the `address`of the person (or smart contract) who called the current function.

{% hint style="info" %}
In Solidity, function execution always needs to start with an external caller. A contract will just sit on the blockchain doing nothing until someone calls one of its functions. So there will always be a `msg.sender`.
{% endhint %}



Here's an example of using `msg.sender` and updating a `mapping`:

```solidity
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // Update our `favoriteNumber` mapping to store `_myNumber` under `msg.sender`
  favoriteNumber[msg.sender] = _myNumber;
  // ^ The syntax for storing data in a mapping is just like with arrays
}

function whatIsMyNumber() public view returns (uint) {
  // Retrieve the value stored in the sender's address
  // Will be `0` if the sender hasn't called `setMyNumber` yet
  return favoriteNumber[msg.sender];
}
```

In this trivial example, anyone could call `setMyNumber`and store a `uint`in our contract, which would be tied to their address. Then when they called `whatIsMyNumber`, they would be returned the uint that they stored.

Using `msg.sender` gives you the security of the Ethereum blockchain — the only way someone can modify someone else's data would be to steal the private key associated with their Ethereum address.
