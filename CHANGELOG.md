# Changelog

All notable changes to this guide are listed here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The repository has no tags or releases.

## [Unreleased]

### Added
- Security section in the README describing what the bridge grants, how to restrict it, and how to shut it off.
- SECURITY, CONTRIBUTING, CODE_OF_CONDUCT, CHANGELOG, issue and pull request templates.
- Troubleshooting rows for the AppleEvents `-1743` send failure and for SSH host-key checks.

### Changed
- README restructured with a header, features and FAQ. The setup steps are unchanged.
- The Python example now quotes the prompt with `shlex.quote`. The old version built the remote command by pasting text into single quotes, so a message containing a quote could run commands on the Mac.
- OpenClaw `remoteHost` guidance: plain `user@host`, with the host key saved in `known_hosts` first.

### Fixed
- The iMessage MCP link in AGENTS.md pointed to a repository that returns 404. It now points to the source listed on npm for `imessage-mcp`.

## 2026-05-16

### Added
- Initial guide: bridge a Linux server AI to iMessage through a Mac over SSH.
- OpenClaw path with `imsg` over an SSH wrapper, and a decision tree (provider-agnostic backend).
- Claude Code with iMessage MCP path for outbound sends, and AGENTS.md for AI agents.
