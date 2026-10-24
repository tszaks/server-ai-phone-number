# Contributing to Server AI Phone Number

Thanks for helping. This repository is a setup guide, not a program. Good changes keep the steps correct, short and safe.

By taking part you agree to follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Before you start

- For a wrong or outdated step, open an issue with the template. Say which step, what you ran and what happened.
- For a new framework or approach, open an issue first. Examples go under `examples/`.
- Small fixes (typos, broken links) can go straight to a pull request.
- Security problems go through [SECURITY.md](SECURITY.md), not a public issue.

## Setup

There is nothing to build. Edit the Markdown files and, where you can, run the commands you changed on a scratch Mac and server before you submit.

## Never include personal data

This is the hard rule for this repository. Never put these in a file, an example, a screenshot, an issue or a pull request:

- Real phone numbers, Apple IDs or email addresses
- Real hostnames, domains or IP addresses
- SSH keys (public or private), API keys or tokens

Use obvious placeholders: `+1XXXXXXXXXX`, `MAC_USERNAME`, `MAC_IP_OR_HOSTNAME`, `SERVER_IP`, `sk-ant-...`.

## Writing style

- **Match the surrounding text.** Short, plain sentences, copy-paste code blocks that work.
- No em dashes. Use a colon, comma or period.
- Every command you add must exist. Link to the upstream docs when you describe another project's behavior.
- Anything that widens access (new `sudo` step, new permission, new SSH option) needs a sentence in the README Security section.

## Pull request checklist

- [ ] Every command and link I added or changed works.
- [ ] No real phone numbers, hostnames, IPs, Apple IDs or keys anywhere in the diff.
- [ ] Security impact of the change is written down if it widens access.
- [ ] README and [AGENTS.md](AGENTS.md) still agree.
- [ ] A line added under `[Unreleased]` in [CHANGELOG.md](CHANGELOG.md) for anything a reader would notice.

## Using AI agents to contribute

Fine, as long as you read and understand what you submit and the checklist above is true. [AGENTS.md](AGENTS.md) tells coding agents how this repository works.

## License

By contributing you agree that your contribution is licensed under the [MIT License](LICENSE).
