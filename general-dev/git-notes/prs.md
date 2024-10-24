# PRs

When a PR is created on GitHub in a forked codebase and that fork allows edits from the main repos maintainers, these are the steps to make changes to that forked branch, which will add the updates to the existing PR.

In the main repo directory, fetch the branch from the remote fork:

```bash
git fetch <FORKED_REPO_URL> <REMOTE_BRANCH_NAME>:<LOCAL_BRANCH_NAME>
git checkout <LOCAL_BRANCH_NAME>
```

{% code title="Example" %}
```bash
git fetch https://github.com/mbonenberger/ethereum-settlers-ui.git marc:marc
git checkout marc
```
{% endcode %}

* Make changes to the branch and commit them to the branch
* Push the changes back to the remote

```bash
git push <FORKED_REPO_URL> <LOCAL_BRANCH_NAME>
```

{% code title="Example" %}
```bash
git push https://github.com/mbonenberger/ethereum-settlers-ui.git marc
```
{% endcode %}

* Then go back to GitHub and review the PR there
