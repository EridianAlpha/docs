# Stack Too Deep

In Solidity, the "stack too deep" error occurs when you have too many local variables in a function. This is a limitation of the Ethereum Virtual Machine (EVM), which Solidity compiles down to. The EVM has a limit on the number of local variables a function can handle, which is typically around 16. This includes explicit local variables, function arguments, and return parameters.

The reason for this limitation is the EVM's design, which uses a stack-based architecture. Each operation in the EVM (including function calls) has a stack to store its data, and this stack has a limited size.

To avoid the "stack too deep" error, you can:

1. **Reduce the number of local variables**: Try to minimize the number of variables in your function. Combine related variables into structs if possible.
2. **Optimize your code**: Sometimes, rearranging the order of variables or breaking a function into smaller functions can help.
3. **Use State Variables**: If some variables can be moved to the contract's state level (outside of the function), this can reduce the number of local variables.
4. **Use Memory Arrays**: For temporary storage of a collection of items, consider using memory arrays.
5. **Inline Functions**: If you're using small helper functions within a larger function, consider inlining these if they are not reused elsewhere.

Be aware that these workarounds might affect gas costs and contract readability, so they should be used judiciously.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p><a href="https://twitter.com/PaulRBerg/status/1612043506545033218">https://twitter.com/PaulRBerg/status/1612043506545033218</a></p></figcaption></figure>
