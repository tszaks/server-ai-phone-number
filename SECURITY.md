# Security policy

## Supported versions

This repository is a guide. Only the current `main` branch is maintained.

## Reporting a vulnerability

Please report security problems privately, not in a public issue.

Use GitHub private vulnerability reporting: open the **Security** tab of this repository and choose **Report a vulnerability**. If that is not available to you, email tyler@szakacsmedia.com.

Please include what you found, what an attacker could do with it, and which step or snippet is involved.

You can expect a first reply within a few days. This repository is maintained by one person, so a fix may take longer, but you will get updates.

## What counts

The setup in this guide gives a server a path into a Mac and into iMessage. Reports about these are welcome:

- **A step or snippet that is unsafe as written.** For example a command that lets text from outside run on your Mac, or a setting that weakens SSH more than the guide says.
- **A missing warning** about something a step grants.
- **Personal data in this repository**: a real phone number, hostname, IP address, Apple ID or key. Report it privately so it can be removed from history.

## What this bridge can access

If you follow this guide, a program on your server can:

1. Log in to your Mac over SSH as the Mac user you choose, and run commands as that user unless you restrict the key.
2. Read all your Messages data (`~/Library/Messages/chat.db` and attachments), because `imsg` needs Full Disk Access.
3. Send iMessage and SMS from your real Apple ID and number, because it has Automation access to Messages.

Anyone who controls that server, or can get text into the AI's input, has the same reach. The [Security section of the README](README.md#security) lists how to reduce the risk and how to shut the bridge off.

This repository does not ship code. Problems in `imsg`, OpenClaw, Claude Code or the iMessage MCP belong in those projects.
