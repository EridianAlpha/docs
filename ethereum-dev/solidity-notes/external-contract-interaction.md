# External Contract Interaction

If a dependency contract had a bug it would render our DApp completely useless — our DApp would point to a hardcoded address that no longer returns the expected result and we'd be unable to modify our contract to fix it.

For this reason, it often makes sense to have functions that will allow you to update key portions of the DApp.

For example, instead of hard coding the contract address into our DApp, we should probably have a setExternalContractAddress function that lets us change this address in the future in case something happens to the external contract being used.

```solidity
ExternalContractInterface externalContract;

function setExternalContractAddress(address _address) external {
    externalContract = ExternalContractInterface(_address);
}
```
