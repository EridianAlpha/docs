# ⚒ Foundry Notes

### Installation

* [https://getfoundry.sh/](https://getfoundry.sh/)

{% code title="Install Foundry" %}
```bash
curl -L https://foundry.paradigm.xyz | bash
```
{% endcode %}

### Initialize New Project

```bash
forge init <PROJECT_NAME>
```

### Install Dependencies

* The command `forge install` is used to install dependencies, such as libraries or other smart contracts that a project may need.

```bash
forge install OpenZeppelin/openzeppelin-contracts@v4.9.3 --no-commit
```

* By default, when you install a dependency using `forge install`, Foundry will automatically create a new Git commit that includes the changes to your project (such as modifications to the `foundry.toml` file and the addition of the OpenZeppelin contract files in your project directory).
* The `--no-commit` flag modifies this behavior. When you use this flag, Foundry will still install the OpenZeppelin contracts, but it will not automatically create a new Git commit for these changes. This means you have to manually commit the changes to your Git repository if you wish to do so.
* The use of `--no-commit` gives you more control over your Git history and commit messages, which can be useful in certain workflows or for maintaining a clean project history.

### Using Foundry

{% content-ref url="useful-commands.md" %}
[useful-commands.md](useful-commands.md)
{% endcontent-ref %}

{% content-ref url="tests/" %}
[tests](tests/)
{% endcontent-ref %}

