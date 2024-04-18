# 📚 Ethereum State Explained

In Ethereum, the entire state of the blockchain, including all accounts and smart contracts, can be thought of as a part of a large, shared state database. Each contract has its own storage, but it's not exactly like a "mapping within a mapping." Let's break this down:

### Ethereum's Global State

* Ethereum's blockchain can be conceptualized as a state machine, where the state includes all accounts (both externally owned accounts and contract accounts) and their balances, nonce, bytecode, and storage (for contract accounts).
* This state is stored in a key-value store (the Ethereum Virtual Machine's state), where each key is an address (20 bytes), and the value is the account's state.

### Contract Storage

* Each smart contract on Ethereum has its own storage space, identified by its contract address. This storage is separate for each contract.
* A contract's storage is a key-value store where both the key and value are 256 bits wide. In this storage, state variables of the contract are stored.
* The way Solidity organizes these storage slots for a contract's state variables is deterministic and follows specific rules, but from the Ethereum protocol's perspective, it just sees a key-value store for each contract.

### Analogy with Mappings

* If we use the analogy of mappings, Ethereum's global state is like a huge mapping where each key is an account address, and the value is the account's state (including a contract's code and storage).
* Within each contract, the storage can be thought of as another mapping, where the keys are essentially the storage slots (sequential for simple variables, computed through hashing for complex types like arrays and mappings) and the values are the contents of those slots.

### Interactions and Independence

* Each contract's storage is independent of others. When a contract is executed, it can only access its own storage directly (though it can call other contracts and trigger changes in their storage through these calls).
* The Ethereum blockchain maintains the integrity and isolation of each contract's storage. This means that a contract cannot directly access or modify the storage of another contract unless explicitly programmed to do so through defined interfaces and function calls.

In summary, Ethereum's state is a large, shared state database, with each contract having its own isolated storage space. This storage space can be conceptualized as a key-value store, unique to each contract, where the contract's state variables are stored.
