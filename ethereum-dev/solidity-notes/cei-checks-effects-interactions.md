# CEI - Checks, Effects, Interactions

Helps to avoid reentrancy and different types of attacks

* Checks
  * Check the criteria for the function
* Effects
  * Changes to the current contract
* Interactions
  * Changes to other contracts

{% tabs %}
{% tab title="Explained" %}
Great StackOverflow answer: [https://ethereum.stackexchange.com/a/56995/106466](https://ethereum.stackexchange.com/a/56995/106466)

**❌ Intuitive approach:**

1. Interact with a contract
2. Check the result.
3. Do something with the result.

**✅ Safe way:**

1. Optimistically deal with state changes (accounting) - assume success.
2. Interact with the contract.
3. Revert changes from step 1 if needed (`.transfer()`) does it automatically. `revert()` is an option, or manually return all values to their previous values in the unlikely event that you want to continue.

It may help to separate contracts into two categories: Those you trust and those you can't be certain about.

There is nothing wrong with, and you often can't avoid calling `view` and `pure` functions in your own system (trusted) and there is nothing wrong with that. You may also want to call `view` functions in other people's contracts that may be suspect - they have made it upgradable, for example, so there is no way ensure the interaction will always be safe even if you see the code today.

Given that those untrusted contracts may themselves inspect the state of _your_ contract, there is a possibility of re-entrant class of exploits. A solution to that problem is to ensure that your state is completely in order before you transfer flow control to the untrusted contract. In other words, don't give them a chance to inspect a half-baked, incomplete transaction in progress because they invoke other functions in your contract or might return with an unexpected result your contract isn't prepared for.

In essence, put your guards up front and gather all the info you will need (checks). Record the complete update to your own state, including the expected results of the final steps, e.g. zero out a balance (effects). Lastly, state-changing operations in other "untrusted" contracts such as `send`, `transfer` or `call`. Notice if it failed and revert your state changes in that case.
{% endtab %}
{% endtabs %}

