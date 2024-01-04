# Interfaces

* Used to create the ABI of a referenced contract without having to actually include all the functions
  * So you say "This contract that I'm referencing has a `fund()` function"
  * This means it can be called, but you don't actually need to know what happens inside the function, you just need to know the name and inputs

{% hint style="info" %}
**CAN NOT:**

* have any functions implemented
* inherit from other contracts, but they can inherit from other interfaces
* declare a constructor
* declare state variables
* declare modifiers



**All declared functions must be external**
{% endhint %}

### Using an Interface

```solidity
interface NumberInterface {
  function getNum(address _myAddress) external view returns (uint);
}

contract MyContract {
  address NumberInterfaceAddress = 0xab38... 
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);

  function someFunction() public {
    uint num = numberContract.getNum(msg.sender);
  }
}
```

In this way, your contract can interact with any other contract on the Ethereum blockchain, as long they expose those functions as `public` or `external`.

### Example

* [https://solidity-by-example.org/interface/](https://solidity-by-example.org/interface/)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;

contract Counter {
    uint public count;

    function increment() external {
        count += 1;
    }
}

interface ICounter {
    function count() external view returns (uint);

    function increment() external;
}

contract MyContract {
    function incrementCounter(address _counter) external {
        ICounter(_counter).increment();
    }

    function getCount(address _counter) external view returns (uint) {
        return ICounter(_counter).count();
    }
}

// Uniswap example
interface UniswapV2Factory {
    function getPair(address tokenA, address tokenB)
        external
        view
        returns (address pair);
}

interface UniswapV2Pair {
    function getReserves()
        external
        view
        returns (
            uint112 reserve0,
            uint112 reserve1,
            uint32 blockTimestampLast
        );
}

contract UniswapExample {
    address private factory = 0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f;
    address private dai = 0x6B175474E89094C44Da98b954EedeAC495271d0F;
    address private weth = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

    function getTokenReserves() external view returns (uint, uint) {
        address pair = UniswapV2Factory(factory).getPair(dai, weth);
        (uint reserve0, uint reserve1, ) = UniswapV2Pair(pair).getReserves();
        return (reserve0, reserve1);
    }
}
```
