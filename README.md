# 📱 Give Your Server AI a Phone Number

> Run your AI on a server? Have a Mac at home? You can give your server AI the ability to send and receive iMessages — your actual phone number, real SMS/iMessage.

---

## How It Works

```
┌─────────────────────┐        SSH        ┌─────────────────────┐
│   Server AI         │ ───────────────▶  │   Your Mac          │
│   (Hermes, Claude,  │                   │   Claude Code CLI   │
│    any LLM agent)   │                   │   + iMessage MCP    │
└─────────────────────┘                   └─────────────────────┘
                                                     │
                                                     ▼
                                            iMessage / SMS
                                          (your real number)
```

Your server AI SSH-es into your Mac. Your Mac runs a Claude Code CLI command with the iMessage MCP. The message goes out from your actual Apple ID — your real phone number.

---

## Prerequisites

- ✅ A Mac (this is the only hard requirement — no Mac, no bridge)
- ✅ A server running your AI (Linux VPS, cloud VM, etc.)
- ✅ An Anthropic API key

---

## Setup: 3 Steps

### Step 1 — Mac: Install Claude Code + iMessage MCP

```bash
# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Install the iMessage MCP globally
npm install -g imessage-mcp

# Set your API key (add to ~/.zshrc or ~/.bash_profile too)
export ANTHROPIC_API_KEY="sk-ant-..."

# Test it — send yourself a message
claude -p "Use the imessage MCP to send a text to +1XXXXXXXXXX saying: test from AI"
```

> **iMessage MCP:** https://github.com/carterjackson/imessage-mcp  
> Grants Claude access to Messages.app. You'll be prompted to allow Full Disk Access for the terminal.

---

### Step 2 — Mac: Enable SSH Access from Your Server

```bash
# On your Mac — enable SSH
sudo systemsetup -setremotelogin on

# On your server — generate a key (if you don't have one)
ssh-keygen -t ed25519 -C "server-ai" -f ~/.ssh/mac_bridge

# Copy the public key to your Mac
ssh-copy-id -i ~/.ssh/mac_bridge.pub your-mac-username@your-mac-ip-or-hostname

# Test it from your server
ssh -i ~/.ssh/mac_bridge your-mac-username@your-mac-ip "echo connected"
```

**Tip:** Use DuckDNS (free) if your Mac's IP changes: https://www.duckdns.org

Add to `~/.ssh/config` on your server:
```
Host mac-bridge
  HostName your-mac-hostname-or-ip
  User your-mac-username
  IdentityFile ~/.ssh/mac_bridge
```

---

### Step 3 — Server: Make Sure SSH Environment Has API Key

By default, SSH sessions don't inherit your shell env. Fix it:

```bash
# On your Mac — create ~/.ssh/environment (OpenSSH reads this for SSH sessions)
echo "ANTHROPIC_API_KEY=sk-ant-..." >> ~/.ssh/environment

# Enable PermitUserEnvironment in sshd_config on your Mac
sudo sh -c 'echo "PermitUserEnvironment yes" >> /etc/ssh/sshd_config'
sudo launchctl stop com.openssh.sshd
sudo launchctl start com.openssh.sshd
```

---

## Usage

Once set up, your server AI can send a text in one command:

```bash
ssh mac-bridge "~/.local/bin/claude -p 'Use the imessage MCP to send a text to +1XXXXXXXXXX saying: Hello from your server AI.' 2>&1"
```

Or with a contact name (iMessage resolves it from your Mac's contacts):

```bash
ssh mac-bridge "~/.local/bin/claude -p 'Use the imessage MCP to send a text to Mom saying: Hey, this is my AI.' 2>&1"
```

---

## Integrating Into Your AI Agent

### Python

```python
import subprocess

def send_imessage(recipient: str, message: str) -> str:
    """Send an iMessage via Mac bridge. recipient = phone number or contact name."""
    prompt = f"Use the imessage MCP to send a text to {recipient} saying: {message}"
    result = subprocess.run(
        ["ssh", "mac-bridge", f"~/.local/bin/claude -p '{prompt}' 2>&1"],
        capture_output=True, text=True, timeout=30
    )
    return result.stdout

# Example
send_imessage("+15551234567", "Your appointment is confirmed for tomorrow at 2pm.")
```

### Hermes Agent (SKILL.md)

If you're running [Hermes Agent](https://github.com/nousresearch/hermes), drop this into your skills:

```yaml
---
name: send-text
description: "Send an iMessage or SMS via Mac bridge."
---

## Workflow
1. Parse recipient and message
2. Confirm before sending
3. Run:

ssh mac-bridge "~/.local/bin/claude -p 'Use the imessage MCP to send a text to [RECIPIENT] saying: [MESSAGE].' 2>&1"

4. Report result
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `claude: command not found` via SSH | Use full path: `~/.local/bin/claude` or `/usr/local/bin/claude` |
| `ANTHROPIC_API_KEY not set` | Add to `~/.ssh/environment` on your Mac (see Step 3) |
| `Permission denied (publickey)` | Run `ssh-copy-id` again; check `~/.ssh/authorized_keys` on Mac |
| iMessage MCP not found | Run `npm install -g imessage-mcp`, then restart Claude |
| Mac IP keeps changing | Use DuckDNS for a stable hostname |
| Full Disk Access error | System Preferences → Privacy → Full Disk Access → add Terminal |

---

## Why This Works (and Why It's Mac-Only)

iMessage is Apple-exclusive. The only way to programmatically send iMessages on a non-jailbroken device is through Messages.app on macOS via AppleScript/SQLite — which is exactly what the iMessage MCP does. There is no API equivalent on Linux or Windows.

If you don't have a Mac, this doesn't apply to you. Consider Twilio for SMS instead.

---

## Security Notes

- Your Mac's SSH port should not be publicly exposed. Use a firewall and only whitelist your server's IP.
- Rotate your SSH keys periodically.
- The `~/.ssh/environment` file stores your API key in plaintext — make sure it's `chmod 600`.
- Never commit your API key anywhere.

---

## Contributing

PRs welcome. If you've got this working with a different AI framework or agent setup, add an integration example.

---

## License

MIT
