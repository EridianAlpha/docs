# Comments (NATSPEC)

* [https://docs.soliditylang.org/en/v0.8.16/natspec-format.html](https://docs.soliditylang.org/en/v0.8.16/natspec-format.html)

### Tags

All tags are optional. The following table explains the purpose of each NatSpec tag and where it may be used. As a special case, if no tags are used then the Solidity compiler will interpret a `///` or `/**` comment in the same way as if it were tagged with `@notice`.

<table data-full-width="true"><thead><tr><th width="189.33333333333331">Tag</th><th width="451">Description</th><th>Context</th></tr></thead><tbody><tr><td><code>@title</code></td><td>A title that should describe the contract/interface</td><td>contract, library, interface</td></tr><tr><td><code>@author</code></td><td>The name of the author</td><td>contract, library, interface</td></tr><tr><td><code>@notice</code></td><td>Explain to an end user what this does</td><td>contract, library, interface, function, public state variable, event</td></tr><tr><td><code>@dev</code></td><td>Explain to a developer any extra details</td><td>contract, library, interface, function, state variable, event</td></tr><tr><td><code>@param</code></td><td>Documents a parameter just like in Doxygen (must be followed by parameter name)</td><td>function, event</td></tr><tr><td><code>@return</code></td><td>Documents the return variables of a contract’s function</td><td>function, public state variable</td></tr><tr><td><code>@inheritdoc</code></td><td>Copies all missing tags from the base function (must be followed by the contract name)</td><td>function, public state variable</td></tr><tr><td><code>@custom:...</code></td><td>Custom tag, semantics is application-defined</td><td>everywhere</td></tr></tbody></table>
