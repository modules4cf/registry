# Contributing to the modules4cf registry

Thank you for helping! This project is built by the community, for the community, and every contribution counts: code, docs, testing, design ideas and reviews.

## Before you start

* **Pick or open an issue first.** Comment on the issue you want to work on so nobody does the same work twice. For bigger changes, agree on the approach in the issue before you write a lot of code.
* Look for [`good first issue`](https://github.com/modules4cf/registry/labels/good%20first%20issue) and [`help wanted`](https://github.com/modules4cf/registry/labels/help%20wanted) if you're new.
* Design decisions (runtime, database, API details) are discussed openly in the issues. Feel free to join in.

## Pull requests

* **One concern per pull request.** A PR fixes one bug, adds one feature or makes one refactoring. Small PRs get reviewed and merged faster.
* **Reference the issue** in the PR description (for example `Closes #12`).
* **Include tests** for every change in behaviour. A bug fix comes with a test that fails without the fix.
* **Tests are never skipped or weakened** to make a build pass. If a test is wrong, fix the test in its own PR and explain why. Tests must not swallow exceptions.
* **Keep CI green.** Every PR runs the build and the test suite.
* **Keep compatibility.** Changes to the `/api/v1` surface must keep the CommandBox CLI working. See [docs/forgebox-api-compatibility.md](docs/forgebox-api-compatibility.md).
* **No secrets in the repository.** Configuration goes through environment variables. Never commit tokens, passwords or keys, not even in tests or examples.
* Update the docs when you change behaviour or configuration.

## Reporting bugs and ideas

Open an [issue](https://github.com/modules4cf/registry/issues) with steps to reproduce, what you expected and what happened, and your CommandBox version if it's relevant.

**Security problems** should not be reported in public issues. Please contact a maintainer privately first.

## Questions

Ask in the issue itself, on the [Lucee Dev Forum](https://dev.lucee.org) or on the [CFML Slack](https://cfml.slack.com).

## License

By contributing, you agree that your contributions are licensed under the [Apache License 2.0](LICENSE).
