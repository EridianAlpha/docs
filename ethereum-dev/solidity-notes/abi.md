# ABI



* [https://coinsbench.com/solidity-tutorial-all-about-abi-46da8b517e7](https://coinsbench.com/solidity-tutorial-all-about-abi-46da8b517e7)
* [https://blog.ricmoo.com/human-readable-contract-abis-in-ethers-js-141902f4d917](https://blog.ricmoo.com/human-readable-contract-abis-in-ethers-js-141902f4d917)



* Hashing functions
  * [https://medium.com/0xcode/hashing-functions-in-solidity-using-keccak256-70779ea55bb0](https://medium.com/0xcode/hashing-functions-in-solidity-using-keccak256-70779ea55bb0)

### ABI with ethers.js

* [https://docs.ethers.io/v4/api-contract.html](https://docs.ethers.io/v4/api-contract.html)

```solidity
// Contract ABI copied directly from Remix
let abi = [
    {
        inputs: [],
        name: "consecutiveWins",
        outputs: [
            {
                internalType: "uint256",
                name: "",
                type: "uint256",
            },
        ],
        stateMutability: "view",
        type: "function",
    },
]

let coinFlipContractAddress = "0xa0714D4539ADcfD7855B811382c49D7185D32977"

// Connect to the Contract
let coinFlipContract = new ethers.Contract(
    coinFlipContractAddress,
    abi,
    rinkebyHttpProvider
)

// Check current consecutiveWins value
let consecutiveWins = await coinFlipContract.consecutiveWins()
```





