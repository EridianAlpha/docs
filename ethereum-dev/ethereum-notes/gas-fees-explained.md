---
icon: gas-pump
---

# Gas Fees Explained

### Gas Limit

Gas limit refers to the maximum amount of gas you are willing to consume on a transaction. More complicated transactions involving smart contracts require more computational work, so they require a higher gas limit than a simple payment. A standard ETH transfer requires a gas limit of 21,000 units of gas.

For example, if you put a gas limit of 50,000 for a simple ETH transfer, the EVM would consume 21,000, and you would get back the remaining 29,000. However, if you specify too little gas, for example, a gas limit of 20,000 for a simple ETH transfer, the EVM will consume your 20,000 gas units attempting to fulfill the transaction, but it will not complete. The EVM then reverts any changes, but since the validator has already done 20k gas units worth of work, that gas is consumed.

{% hint style="info" %}
The gas limit is like the distance you want to travel in a car. E.g. if my journey is 21km away (a standard transfer) then I must have at minimum 21km worth of gas in my car. If I get to the destination without using the whole tank of gas (because I specified a higher limit than I needed) then I get to keep the gas in the car (ETH gets refunded to my wallet).
{% endhint %}

### Max Fee

{% hint style="info" %}
Max Fee is the maximum amount you are willing to pay for the gas at the pump at the start of the journey. So if you don't mind when you start the journey (e.g. the middle of the night) then you can wait until gas is cheap, and your journey will only start when the max fee lowers to your desired amount (transaction will only be picked up at that point, but not necessarily executed because of... Max Priority Fee).
{% endhint %}

### Max Priority Fee

{% hint style="info" %}
Once gas reaches the price you want to pay for your journey, at that point there might be a long queue at the gas station for you to fill up, and while you're waiting in line the price could go up again! So a priority fee is like a bribe/tip you pay to the pump attendant to fill up your car first before anyone else.
{% endhint %}

### Max Transaction Fee

If you are buying 21,000 units of gas, and you set a `max fee` to limit the maximum amount you will pay, then the validator will pocket the difference from the base fee, up to the `max priority fee` set. So, if I want to buy 21,000 units of gas, and I set a `max fee` of 50 Gwei and a `max priority fee` of 5 Gwei, then if the `base fee` at the time of the transaction is:

* 47 Gwei then the validator will earn the 3 Gwei.
  * But if 3 Gwei isn't a high enough fee compared with other transactions then I'll be left out.
* 45 Gwei then the validator will earn the maximum 5 Gwei.
  * Again, only if 5 is enough of a tip to beat other people in the same block.
* 40 Gwei then the validator will still only earn a maximum of 5 Gwei as that's the limit I set and any difference will be returned to the senders wallet.
  * This returned amount will be a messy number.
