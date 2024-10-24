# ❔ Interview Questions

{% embed url="https://www.rareskills.io/post/solidity-interview-questions" %}

## 1. Easy Questions

All of these questions can be answered in three sentences or less.

### 1.1. What is the difference between private, internal, public, and external functions?



### 1.2. Approximately, how large can a smart contract be?



### 1.3. What is the difference between create and create2?



### 1.4. What is the difference between create and create2?



### 1.5. What major change with arithmetic happened with Solidity 0.8.0?



### 1.6. What special CALL is required for proxies to work?



### 1.7. How do you calculate the dollar cost of an Ethereum transaction?



### 1.8. What are the challenges of creating a random number on the blockchain?



### 1.9. What is the difference between a Dutch Auction and an English Auction?



### 1.10. What is the difference between transfer and transferFrom in ERC20?



### 1.11. Which is better to use for an address allowlist: a mapping or an array?



### 1.12. Why? Why shouldn’t tx.origin be used for authentication?



### 1.13. What hash function does Ethereum primarily use?



### 1.14. How much is 1 gwei of Ether? How much is 1 wei of Ether?



### 1.15. What is the difference between assert and require?



### 1.16. What is a flash loan? What is the check-effects-interaction pattern?



### 1.17. What is the minimum amount of Ether required to run a solo staking node?



### 1.18. What is the difference between fallback and receive?



### 1.19. What is reentrancy?



### 1.20. What prevents infinite loops from running forever?



### 1.21. What is the difference between tx.origin and msg.sender?



### 1.22. How do you send Ether to a contract that does not have payable functions, or a receive or fallback?



### 1.22. What is the difference between view and pure?



### 1.23. What is the difference between transferFrom and safeTransferFrom in ERC721?



### 1.24. How can an ERC1155 token be made into a non-fungible token?



### 1.25. What is access control and why is it important? What does a modifier do?



### 1.26. What is the largest value a uint256 can store?



### 1.27. What is variable and fixed interest rate?



## 2. Medium

### 2.1 What is the difference between transfer and send?



### 2.2 Why should they not be used?



### 2.3 What is a storage collision in a proxy contract?



### 2.4 What is the difference between abi.encode and abi.encodePacked?



### 2.5 uint8, uint32, uint64, uint128, uint256 are all valid uint sizes. Are there others?



### 2.6 What changed with block.timestamp before and after proof of stake?



### 2.7 What is frontrunning?



### 2.8 What is a commit-reveal scheme and when would you use it?



### 2.9 Under what circumstances could abi.encodePacked create a vulnerability?



### 2.10 How does Ethereum determine the BASEFEE in EIP-1559?



### 2.11 What is the difference between a cold read and a warm read?



### 2.12 How does an AMM price assets?



### 2.13 What is a function selector clash in a proxy and how does it happen?



### 2.14 What is the effect on gas of making a function payable?



### 2.15 What is a signature replay attack?



### 2.16 How would you design a game of rock-paper-scissors in a smart contract such that players cannot cheat?



### 2.17 What is the free memory pointer and where is it stored?



### 2.18 What function modifiers are valid for interfaces?



### 2.19 What is the difference between memory and calldata in a function argument?



### 2.20 Describe the three types of storage gas costs for writes.



### 2.21 Why shouldn’t upgradeable contracts use the constructor?



### 2.22 What is the difference between UUPS and the Transparent Upgradeable Proxy pattern?



### 2.23 If a contract delegatecalls an empty address or an implementation that was previously self-destructed, what happens?



### 2.24 What if it is a low-level call instead of a delegatecall?



### 2.25 What danger do ERC777 tokens pose?



### 2.26 According to the solidity style guide, how should functions be ordered?



### 2.27 According to the solidity style guide, how should function modifiers be ordered?



### 2.28 What is a bonding curve?



### 2.29 How does \_safeMint differ from \_mint in the OpenZeppelin ERC721 implementation?



### 2.30 What keywords are provided in Solidity to measure time?



### 2.31 What is a sandwich attack?



### 2.32 If a delegatecall is made to a function that reverts, what does the delegatecall do?



### 2.33 What is a gas efficient alternative to multiplying and dividing by a power of two?



### 2.34 How large a uint can be packed with an address in one slot?



### 2.35 Which operations give a partial refund of gas?



### 2.36 What is ERC165 used for?



### 2.37 If a proxy makes a delegatecall to A, and A does address(this).balance, whose balance is returned, the proxy’s or A?



### 2.38 What is a slippage parameter useful for?



### 2.39 What does ERC721A do to reduce mint costs? What is the tradeoff?



### 2.40 Why doesn’t Solidity support floating point arithmetic?



### 2.41 What is TWAP? How does Compound Finance calculate utilization?



### 2.42 If a delegatecall is made to a function that reads from an immutable variable, what will the value be?



### 2.43 What is a fee-on-transfer token?



### 2.44 What is a rebasing token?



### 2.45 In what year will a timestamp stored in a uint32 overflow?



### 2.46 What is LTV in the context of DeFi?



### 2.47 What are aTokens and cTokens in the context of Compound Finance and AAVE?



### 2.48 Describe how to use a lending protocol to go leveraged long or leveraged short on an asset.



### 2.49 What is a perpetual protocol?



### Hard

1. How does fixed point arithmetic represent numbers?
2. What is an ERC20 approval frontrunning attack?
3. What opcode accomplishes address(this).balance?
4. How many arguments can a solidity event have?
5. What is an anonymous Solidity event?
6. Under what circumstances can a function receive a mapping as an argument?
7. What is an inflation attack in ERC4626
8. How many storage slots does this use? `uint64[] x = [1,2,3,4,5]`? Does it differ from memory?
9. Prior to the Shanghai upgrade, under what circumstances is `returndatasize()` more efficient than `PUSH 0`?
10. Why does the compiler insert the INVALID op code into Solidity contracts?
11. What is the difference between how a custom error and a require with error string is encoded at the EVM level? 1hat is the kink parameter in the Compound DeFi formula? 1ow can the name of a function affect its gas cost, if at all?
12. What is a common vulnerability with ecrecover?
13. What is the difference between an optimistic rollup and a zk-rollup?
14. How does EIP1967 pick the storage slots, how many are there, and what do they represent?
15. How much is one Sazbo of ether?
16. What can delegatecall be used for besides use in a proxy?
17. Under what circumstances would a smart contract that works on Etheruem not work on Polygon or Optimism? (Assume no dependencies on external contracts)
18. How can a smart contract change its bytecode without changing its address?
19. What is the danger of putting msg.value inside of a loop? escribe the calldata of a function that takes a dynamic length array of `uint128` when `uint128[1,2,3,4]` is passed as an argument
20. Why is strict inequality comparisons more gas efficient than ≤ or ≥? What extra opcode(s) are added?
21. If a proxy calls an implementation, and the implementation self-destructs in the function that gets called, what happens?
22. What is the relationship between variable scope and stack depth?
23. What is an access list transaction?
24. How can you halt an execution with the mload opcode?
25. What is a beacon in the context of proxies?
26. Why is it necessary to take a snapshot of balances before conducting a governance vote?
27. How can a transaction be executed without a user paying for gas?
28. In solidity, without assembly, how do you get the function selector of the calldata?
29. How is an Ethereum address derived?
30. What is the metaproxy standard?
31. If a try catch makes a call to a contract that does not revert, but a revert happens inside the try block, what happens?
32. If a user calls a proxy makes a delegatecall to A, and A makes a regular call to B, from A’s perspective, who is `msg.sender`? from B’s perspective, who is `msg.sender`? From the proxy’s perspective, who is `msg.sender`?
33. Under what circumstances do vanity addresses (leading zero addresses) save gas?
34. Why do a significant number of contract bytecodes begin with 6080604052? What does that bytecode sequence do?
35. How does Uniswap V3 determine the boundaries of liquidity intervals?
36. What is the risk-free rate?
37. When a contract calls another call via call, delegatecall, or staticcall, how is information passed between them?
38. What is the difference between `bytes` and `bytes1[]`?
39. What is the most amount of leverage that can be achieved in a borrow-swap-supply-collateral loop if the LTV is 75%? What about other LTV limits?
40. How does Curve StableSwap achieve concentrated liquidity?
41. What quirks does the Tether stablecoin contract have?
42. What is the smallest uint that will store 1 million? 1 billion? 1 trillion? 1 quadrillion?
43. What danger to uninitialized UUPS logic contracts pose?
44. What is the difference (if any) between what a contract returns if a divide-by-zero happens in Soliidty or if a dividye-by-zero happens in Yul?
45. Why can’t `.push()` be used to append to an array in memory?

### Advanced

1. What addresses to the ethereum precompiles live at?
2. Describe what “liquidity” is in the context of Uniswap V2 and Uniswap V3.
3. If a delegatecall is made to a contract that makes a delegatecall to another contract, who is msg.sender in the proxy, the first contract, and the second contract?
4. What is the difference between how a `uint64` and `uint256` are abi-encoded in calldata?
5. What is read-only reentrancy?
6. What are the security considerations of reading a (memory) bytes array from an untrusted smart contract call?
7. If you deploy an empty Solidity contract, what bytecode will be present on the blockchain, if any?
8. How does the EVM price memory usage?
9. What is stored in the metadata section of a smart contract?
10. What is the uncle-block attack from an MEV perspective?
11. How do you conduct a signature malleability attack?
12. Under what circumstances do addresses with leading zeros save gas and why?
13. What is the difference between `payable(msg.sender).call{value: value}("")` and `msg.sender.call{value: value}("")`? 1How many storage slots does a string take up?
14. How does the `--via-ir` functionality in the Solidity compiler work?
15. Are function modifiers called from right to left or left to right, or is it non-deterministic?
16. If you do a delegatecall to a contract and the opcode CODESIZE executes, which contract size will be returned?
17. Why is it important to ECDSA sign a hash rather than an arbitrary bytes32?
18. Describe how symbolic manipulation testing works.
19. What is the most efficient way to copy regions of memory?
20. How can you validate on-chain that another smart contract emitted an event, without using an oracle?
21. When selfdestruct is called, at what point is the Ether transferred? At what point is the smart contract’s bytecode erased?
22. Under what conditions does the Openzeppelin Proxy.sol overwrite the free memory pointer? Why is it safe to do this?
23. Why did Solidity deprecate the “years” keyword?
24. What does the verbatim keyword do, and where can it be used?
25. How much gas can be forwarded in a call to another smart contract?
26. What does an int256 variable that stores -1 look like in hex?
27. What is the use of the signextend opcode?
28. Why do negative numbers in calldata cost more gas?
29. What is a zk-friendly hash function and how does it differ from a non-zk-friendly hash function?
30. What does a metaproxy do?
31. What is a nullifier in the context of zero knowledge, and what is it used for?
32. What is SECP256K1?
33. Why shouldn’t you get price from `slot0` in Uniswap V3?
34. Describe how to compute the 9th root of a number on-chain in Solidity.
35. What is the danger of using return in assembly out of a Solidity function that has a modifier?
36. Without using the `%` operator, how can you determine if a number is even or odd?
37. What does `codesize()` return if called within the constructor? What about outside the constructor?
