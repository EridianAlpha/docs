# Security

* [https://docs.openzeppelin.com/contracts/4.x/api/security](https://docs.openzeppelin.com/contracts/4.x/api/security)
  * These contracts aim to cover common security practices

### PullPayment

* A pattern that can be used to avoid reentrancy attacks

### ReentrancyGuard

* [https://solidity-by-example.org/hacks/re-entrancy/](https://solidity-by-example.org/hacks/re-entrancy/)
* A modifier that can prevent reentrancy during certain functions

```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract FundMe is ReentrancyGuard {
...

    function withdraw() external payable onlyOwner nonReentrant {
...
```

### Pausable

* A common emergency response mechanism that can pause functionality while a remediation is pending

### Constructor

* Older versions of Solidity didn't have the `constructor` keyword
* It took the function with _EXACTLY_  the same name as the contract and used that as the constructor
* So... if you got the name wrong even a little bit, the constructor wouldn't run and become accessible to anyone
* Rubixi bug: [https://www.youtube.com/watch?v=h4dxwYQQ\_b8](https://www.youtube.com/watch?v=h4dxwYQQ\_b8)

### Self Destruct

* [https://solidity-by-example.org/hacks/self-destruct/](https://solidity-by-example.org/hacks/self-destruct/)
* If you send funds then immediately call `selfdestruct()` then the contract you call could try to revert and send the funds back, but it can't since you selfdestructed, so it keeps the funds, but doesn't continue with the code past the revert point
* If you code your contract badly, this could brick the function (e.g. setting a winner after checking the current balance)
* Therefore, use a counter that is part of the function, instead of `address(this).balance` so that the code is executed as expected during revert

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

// The goal of this game is to be the 7th player to deposit 1 Ether.
// Players can deposit only 1 Ether at a time.
// Winner will be able to withdraw all Ether.

/*
1. Deploy EtherGame
2. Players (say Alice and Bob) decides to play, deposits 1 Ether each.
2. Deploy Attack with address of EtherGame
3. Call Attack.attack sending 5 ether. This will break the game
   No one can become the winner.

What happened?
Attack forced the balance of EtherGame to equal 7 ether.
Now no one can deposit and the winner cannot be set.
*/

contract EtherGame {
    uint256 public targetAmount = 7 ether;
    address public winner;

    function deposit() public payable {
        require(msg.value == 1 ether, "You can only send 1 Ether");

        uint balance = address(this).balance;
        require(balance <= targetAmount, "Game is over");

        // ********************
        // THIS IS THE PROBLEM
        // Winner is set after the check, but the check fails and reverts, so the winner is never set
        // even though the value of address(this).balance is now greater than 7
        // ********************    
        if (balance == targetAmount) {
            winner = msg.sender;
        }
    }

    function claimReward() public {
        require(msg.sender == winner, "Not winner");

        (bool sent, ) = msg.sender.call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
    }
}

contract Attack {
    EtherGame etherGame;

    constructor(EtherGame _etherGame) {
        etherGame = EtherGame(_etherGame);
    }

    function attack() public payable {
        // You can simply break the game by sending ether so that
        // the game balance >= 7 ether

        // cast address to payable
        address payable addr = payable(address(etherGame));
        selfdestruct(addr);
    }
}
```

* So instead, use a global balance variable to keep track of the funds, not just the contract balance

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract EtherGame {
    uint256 public targetAmount = 3 ether;
    uint256 public balance;
    address public winner;

    function deposit() public payable {
        require(msg.value == 1 ether, "You can only send 1 Ether");

        balance += msg.value;
        require(balance <= targetAmount, "Game is over");

        if (balance == targetAmount) {
            winner = msg.sender;
        }
    }

    function claimReward() public {
        require(msg.sender == winner, "Not winner");

        (bool sent, ) = msg.sender.call{value: balance}("");
        require(sent, "Failed to send Ether");
    }
}
```
