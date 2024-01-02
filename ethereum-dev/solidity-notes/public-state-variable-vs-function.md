# Public State Variable vs Function

### **Using a Function**

```solidity
solidityCopy codefunction name() public pure returns (string memory) {
    return "Manual Token";
}
```

This is a function that, when called, returns the string "Manual Token". Each time this function is called, it consumes gas because it's a transaction on the Ethereum network. However, since it's a `pure` function (not reading from or modifying the state), the gas cost is relatively low. But it's important to note that every call to this function is a read operation on the blockchain.

### **Using a Public State Variable**

```solidity
solidityCopy codestring public name = "Manual Token";
```

This approach declares `name` as a public state variable. In Solidity, public state variables automatically have a getter function created by the compiler. This means that when `name` is accessed, it's done through an auto-generated function that's similar to the manually written function in the first approach.

The key difference is that reading this public state variable is a read-only operation and does not consume gas if called externally (i.e., outside of a transaction). However, the initial deployment of the contract will cost slightly more gas because this variable is stored on the blockchain as part of the contract's state.

In terms of gas efficiency for reads:

* If you're only concerned about the cost of reading the value (and not the deployment cost), using the public state variable (`string public name = "Manual Token";`) is more gas-efficient since it doesn't cost gas to read from a public state variable.
* If considering the cost of contract deployment, the function approach might use slightly less gas at deployment time, but this is a one-time cost.

In summary, for most practical purposes, especially if the `name` will be read frequently, using the public state variable is the more gas-efficient approach due to free external reads.
