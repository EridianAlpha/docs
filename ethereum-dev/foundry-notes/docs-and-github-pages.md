---
icon: file-lines
---

# Docs & GitHub Pages

Foundry has a built-in documentation feature that generates an `mdbook` for all contracts in the `src` directory. NATSPEC comments are used to populate the content. This can then be built using GitHub Actions and hosted using GitHub pages.

The default configuration is ok, but I've customized it with a GitHub action which on every push to the repo:

1. Builds the updated docs.
2. Customizes the config.
3. Commits the changes to the documentation branch `gh-pages`.
4. Publishes the updated docs on GitHub pages.

## Viewing Locally

To view the site locally run:

```bash
forge doc --build --serve --port=4000
```

* This won't have the config customizations made using the GitHub Actions workflow, but it can be useful for local development.

## GitHub Actions Workflow

* Create a workflow `.yml` file at `.github/workflows/deployGitHubPages.yml`

{% hint style="info" %}
Modify:

* \<ADD\_TITLE\_HERE>
* \<ADD\_AUTHOR\_HERE>
{% endhint %}

{% code title=".github/workflows/deployGitHubPages.yml" fullWidth="true" %}
```yaml
name: Deploy Docs to GitHub Pages

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Install Foundry
        uses: foundry-rs/foundry-toolchain@v1

      - name: Install Dependencies
        run: forge install

      - name: Generate Documentation
        run: forge doc

      - name: Edit book.toml
        run: |
          sed -i 's/title = ""/title = "<ADD_TITLE_HERE>"/' ./docs/book.toml                          # Set the title of the book
          sed -i 's/authors = \[\]/authors = \["<ADD_AUTHOR_HERE>"\]/' ./docs/book.toml               # Set the author of the book
          sed -i '/\[book\]/a language = "en"' ./docs/book.toml                                       # Add language setting under [book]
          sed -i '/\[book\]/a multilingual = false' ./docs/book.toml                                  # Add multilingual setting under [book]
          sed -i 's/no-section-label = true/no-section-label = false/' ./docs/book.toml               # Change no-section-label to false
          sed -i '/\[output.html\]/a default-theme = "dark"' ./docs/book.toml                         # Add default-theme under [output.html]
          sed -i '/\[output.html\]/a preferred-dark-theme = "ayu"' ./docs/book.toml                   # Add preferred-dark-theme under [output.html]
          sed -i '/^\[output.html.fold\]$/,/^\[/ s/^enable = true/enable = false/' ./docs/book.toml   # Change enable under [output.html.fold]

      - name: Edit SUMMARY.md
        run: |
          sed -i 's/❱ //g' ./docs/src/SUMMARY.md                                        # Removes the "❱ " from lines
          sed -i '/^# src$/d' ./docs/src/SUMMARY.md                                     # Deletes the line containing exactly "# src"
          sed -i 's/- \[Home\](README.md)/[README](README.md)/' ./docs/src/SUMMARY.md   # Replaces "- [Home](README.md)" with "[README](README.md)"

      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable

      - name: Install mdbook
        run: |
          mkdir mdbook
          curl -sSL https://github.com/rust-lang/mdBook/releases/download/v0.4.36/mdbook-v0.4.36-x86_64-unknown-linux-gnu.tar.gz | tar -xz --directory=./mdbook
          echo `pwd`/mdbook >> $GITHUB_PATH

      - name: Build book
        run: |
          cd ./docs
          mdbook build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs/book
```
{% endcode %}

### GitHub Pages Repo Settings

{% hint style="warning" %}
The branch `gh-pages` is created by the Actions workflow, so this step can only be completed after the Action has run successfully for the first time.
{% endhint %}

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure></div>
