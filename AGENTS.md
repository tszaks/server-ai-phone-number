# AGENTS.md — LLM Instructions for This Repo

> This file tells AI agents (Claude, Copilot, Cursor, OpenClaw, etc.) exactly what this repo does and how to help users set it up.

---

## What This Repo Does

This repo teaches how to bridge a **Linux server AI** to **iMessage** via a Mac, using one of two approaches:

1. **OpenClaw** (full gateway with `imsg` CLI) — two-way iMessage, officially supported remote Mac path
2. **Claude Code + iMessage MCP** — simple outbound-only sends from any existing AI agent

The Mac is the mandatory bridge in both cases. No Mac = no iMessage.

---

## When Helping a User Set This Up

### First: Verify Prerequisites

Ask or determine:
1. Do they have a Mac signed into iMessage? → **Required.** If no, suggest Twilio and stop.
2. Do they have a server (Linux VPS, VM, etc.)?
3. What do they want: full two-way AI conversations (→ OpenClaw) or just outbound sends (→ Claude Code + iMessage MCP)?

---

## Option A: OpenClaw Setup (Full iMessage)

**Official docs:** https://docs.openclaw.ai/channels/imessage  
**Repo:** https://github.com/openclaw/openclaw

This is OpenClaw's documented remote Mac path. The gateway runs on Linux; `cliPath` points to a shell wrapper that SSHes to the Mac and runs `imsg` there.

### Mac Steps

```bash
# Install imsg
brew install steipete/tap/imsg
imsg --version
imsg status --json

# Run once interactively to trigger macOS permission prompts
imsg chats --limit 1

# Enable SSH
sudo systemsetup -setremotelogin on
```

**Required macOS permissions:** Full Disk Access + Automation for the terminal/process running `imsg`.

### Server Steps

```bash
# Install OpenClaw (Node 20+ required)
npx openclaw onboard   # interactive wizard (recommended)
# OR
npm install -g openclaw

# Generate SSH key and copy to Mac
ssh-keygen -t ed25519 -C "ai-mac-bridge" -f ~/.ssh/mac_bridge
ssh-copy-id -i ~/.ssh/mac_bridge.pub USERNAME@MAC_IP

# Add to ~/.ssh/config
cat >> ~/.ssh/config << 'EOF'
Host mac-bridge
  HostName MAC_IP_OR_HOSTNAME
  User MAC_USERNAME
  IdentityFile ~/.ssh/mac_bridge
EOF

# Create SSH wrapper script
mkdir -p ~/.openclaw/scripts
cat > ~/.openclaw/scripts/imsg-ssh << 'EOF'
#!/usr/bin/env bash
exec ssh -T mac-bridge imsg "$@"
EOF
chmod +x ~/.openclaw/scripts/imsg-ssh

# Test the wrapper
~/.openclaw/scripts/imsg-ssh chats --limit 1
```

### OpenClaw Config

```json
{
  "channels": {
    "imessage": {
      "enabled": true,
      "cliPath": "~/.openclaw/scripts/imsg-ssh",
      "remoteHost": "USERNAME@MAC_IP",
      "includeAttachments": true
    }
  }
}
```

### Start + Pair

```bash
openclaw gateway start

# Approve first incoming DM:
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

---

## Option B: Claude Code + iMessage MCP (Outbound Only)

For agents that need to send texts without a full OpenClaw gateway.

### Mac Steps

```bash
npm install -g @anthropic-ai/claude-code
npm install -g imessage-mcp

export ANTHROPIC_API_KEY="sk-ant-..."
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc

# Allow API key in SSH sessions
echo "ANTHROPIC_API_KEY=sk-ant-..." >> ~/.ssh/environment
sudo sh -c 'echo "PermitUserEnvironment yes" >> /etc/ssh/sshd_config'
sudo launchctl stop com.openssh.sshd && sudo launchctl start com.openssh.sshd

sudo systemsetup -setremotelogin on
```

Grant **Full Disk Access** to Terminal.app: System Preferences → Privacy & Security → Full Disk Access.

### Server Steps

```bash
ssh-keygen -t ed25519 -C "ai-mac-bridge" -f ~/.ssh/mac_bridge
ssh-copy-id -i ~/.ssh/mac_bridge.pub USERNAME@MAC_IP

cat >> ~/.ssh/config << 'EOF'
Host mac-bridge
  HostName MAC_IP
  User MAC_USERNAME
  IdentityFile ~/.ssh/mac_bridge
EOF
```

### One-Liner Test

```bash
ssh mac-bridge "~/.local/bin/claude -p 'Use the imessage MCP to send a text to +1XXXXXXXXXX saying: bridge is live.' 2>&1"
```

If that sends a text, the bridge is working.

### Python Integration

```python
import subprocess

def send_imessage(recipient: str, message: str) -> str:
    prompt = f"Use the imessage MCP to send a text to {recipient} saying: {message}"
    result = subprocess.run(
        ["ssh", "mac-bridge", f"~/.local/bin/claude -p '{prompt}' 2>&1"],
        capture_output=True, text=True, timeout=30
    )
    return result.stdout
```

---

## Decision Tree

```
User wants iMessage on server AI?
├── Has a Mac? → No → Suggest Twilio, stop.
└── Has a Mac? → Yes
    ├── Needs two-way conversations / receive messages? → OpenClaw (Option A)
    └── Just needs to send texts from existing agent? → Claude Code + MCP (Option B)
```

---

## Common Failures

| Symptom | Fix |
|---|---|
| `imsg: not found` via SSH | Install: `brew install steipete/tap/imsg`. Use full path. |
| `Full Disk Access` error | System Prefs → Privacy → Full Disk Access → add Terminal |
| `claude: not found` via SSH | Find with `ssh mac-bridge "which claude"`, use full path |
| `ANTHROPIC_API_KEY not set` in SSH | Add to `~/.ssh/environment`, enable `PermitUserEnvironment yes` |
| `Permission denied (publickey)` | Re-run `ssh-copy-id`, check `authorized_keys` on Mac |
| Mac IP changes | Use DuckDNS: https://www.duckdns.org |
| OpenClaw pairing expired | Pairing expires in 1h — trigger a new DM to get a fresh code |
| Attachments broken in OpenClaw | Set `remoteHost` and `remoteAttachmentRoots` in config |

---

## What Not to Do

- Do not commit API keys
- Do not expose Mac SSH publicly without IP whitelisting
- Do not run untrusted prompts through the bridge (it has Messages.app access)
- Do not suggest workarounds before trying the documented path first
- If user has no Mac: do not attempt to find alternatives that claim to work — they won't. Suggest Twilio.

---

## Repo Structure

```
server-ai-phone-number/
├── README.md       ← Human-readable setup guide (both options)
├── AGENTS.md       ← This file (LLM-optimized instructions)
└── LICENSE
```

---

## External References

- OpenClaw repo: https://github.com/openclaw/openclaw
- OpenClaw iMessage docs: https://docs.openclaw.ai/channels/imessage
- imsg CLI: `brew install steipete/tap/imsg`
- iMessage MCP: https://github.com/carterjackson/imessage-mcp
- DuckDNS: https://www.duckdns.org
- Twilio (no-Mac SMS alternative): https://twilio.com
