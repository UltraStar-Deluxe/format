# Contributing Guide

We're open for Pull Requests.
Our endgoal is to bond every community out there so we'd love to get in touch with you and partake in discussions on either [Issues](https://github.com/UltraStar-Deluxe/format/issues), [PR's](https://github.com/UltraStar-Deluxe/format/pulls) or [Discord - Format Related channel](https://discord.gg/tNEXZw2QJX)

## Setup your IDE

If you want to contribute please setup your IDE or Editor correctly.
To do this you'll have to make sure the Editorconfig is used (works per default in VSCode) and that you have `pre-commit` installed so your commits are checked before pushing.

### Install & configure pre-commit (through python)

Please execute the following commands within the repositories root directory.
This will install pre-commit to your system and installs the git hooks which check your changed code on commit if your changes adhere the conventions.

1. `pip install pre-commit`
2. `pre-commit install`

### Linting

We want to ensure a consistent Markdown code style across this repository.
All markdown files will be linted via [`markdownlint-cli2-action`](https://github.com/DavidAnson/markdownlint-cli2-action).
You can run the linter locally by installing [`markdownlint-cli2`](https://github.com/DavidAnson/markdownlint-cli2) and then running

```shell
markdownlint-cli2 "**/*.md" "#docs" "#.github/PULL_REQUEST_TEMPLATE.md"
```

We currently exclude the `docs` folder and the pull request template from the Markdown linting.
