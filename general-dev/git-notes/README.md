# 💾 Git Notes

```
Still to document
- PRs (https://www.git-scm.com/docs/git-request-pull)
- Conflict resolution
- Squashing
- Options during an interactive rebase
```

### Commands

* [repos.md](repos.md "mention")
* [committing-changes.md](committing-changes.md "mention")
* [branches.md](branches.md "mention")
* [merging-and-rebasing.md](merging-and-rebasing.md "mention")
* [prs.md](prs.md "mention")

### Useful Resources

* Git learning game: [https://learngitbranching.js.org](https://learngitbranching.js.org/)
* Useful commands for fixing mistakes: [https://ohshitgit.com](https://ohshitgit.com)

### Connecting to GitHub

* Logging in to GitHub on the command line: [https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
  * Use a personal access token instead of a password
  * [https://stackoverflow.com/questions/35942754/how-can-i-save-username-and-password-in-git](https://stackoverflow.com/questions/35942754/how-can-i-save-username-and-password-in-git)
  * View stored credentials (plain text)
    * `vim ~/.git-credentials`
* Using SSH Key
  * [https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

### Git Configuration Settings

* View/edit git configuration settings
  * If there's nothing there already this will create a new file

```bash
vim ~/.gitconfig
```

* Add this to the `[alias]` section
  * Pretty view of commit tree for current branch

{% code overflow="wrap" fullWidth="false" %}
```bash
[alias] tree = log --color --graph --pretty=format:'%Cred%h%Creset -%C(bold magenta)%d%Creset %s %C(cyan)(%cr)%C(bold blue) <%an> %Creset' --date=relative --abbrev-commit --all

[alias] tree-current = log --color --graph --pretty=format:'%Cred%h%Creset -%C(bold magenta)%d%Creset %s %C(cyan)(%cr)%C(bold blue) <%an> %Creset' --date=relative --abbrev-commit --all
```
{% endcode %}

* Then run the commands

```bash
git tree
git tree-current
```
