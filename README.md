# 📱 Give Your Server AI a Phone Number

> Run your AI on a server? Have a Mac at home? You can give your server AI the ability to send and receive iMessages — your actual phone number, real SMS/iMessage.

Two supported approaches:

| Approach | Tool | Best for |
|---|---|---|
| **OpenClaw** (recommended) | `imsg` CLI + OpenClaw gateway | Full iMessage integration — receive, reply, react, groups |
| **Claude Code + iMessage MCP** | `claude` CLI + MCP | Simple outbound-only sends from any AI agent |

---

## How It Works

```
┌─────────────────────┐        SSH        ┌─────────────────────┐
│   Server AI         │ ───────────────▶  │   Your Mac          │
│   (OpenClaw,        │                   │   imsg CLI          │
│    Hermes, custom)  │                   │   Messages.app      │
└─────────────────────┘                   └─────────────────────┘
                                                     │
                                                     ▼
                                            iMessage / SMS
                                          (your real number)
```

Your server AI connects to your Mac via SSH. The Mac runs `imsg` (or Claude Code + iMessage MCP) to send/receive messages through Messages.app — your actual Apple ID, your real phone number.

---

## Prerequisites

- ✅ A Mac signed into iMessage (this is the only hard requirement — no Mac, no bridge)
- ✅ A server running your AI (Linux VPS, cloud VM, etc.)
- ✅ SSH access from server to Mac

---

## Option A: OpenClaw (Recommended — Full iMessage)

OpenClaw is an open source personal AI assistant with native iMessage support via `imsg`. When your gateway runs on Linux, you point it at a wrapper script that SSHes to your Mac and runs `imsg` there. This is OpenClaw's officially documented remote Mac path.

**What you get:** Inbound + outbound messages, reactions, threaded replies, group chats, attachments. Full two-way integration.

**Source:** [openclaw/openclaw](https://github.com/openclaw/openclaw) · [iMessage channel docs](https://docs.openclaw.ai/channels/imessage)

### Step 1 — Mac: Install imsg

```bash
brew install steipete/tap/imsg

# Verify
imsg --version
imsg status --json

# Grant permissions (run these once interactively to trigger macOS prompts)
# Full Disk Access + Automation required — approve in System Preferences when prompted
imsg chats --limit 1
```

### Step 2 — Mac: Enable SSH

```bash
sudo systemsetup -setremotelogin on
```

### Step 3 — Server: Install OpenClaw

```bash
# Requires Node.js 20+
npx openclaw onboard
# Follow the interactive wizard — it walks through gateway, workspace, and channel setup
```

Or manual install:

```bash
npm install -g openclaw
openclaw --version
```

### Step 4 — Server: Create SSH wrapper script

OpenClaw's remote Mac path works by pointing `cliPath` at a wrapper that SSHes to your Mac:

```bash
mkdir -p ~/.openclaw/scripts

cat > ~/.openclaw/scripts/imsg-ssh << 'EOF'
#!/usr/bin/env bash
exec ssh -T mac-bridge imsg "$@"
EOF

chmod +x ~/.openclaw/scripts/imsg-ssh
```

Add `mac-bridge` to `~/.ssh/config` on your server:

```
Host mac-bridge
  HostName your-mac-hostname-or-ip
  User your-mac-username
  IdentityFile ~/.ssh/mac_bridge
```

Test the wrapper:

```bash
~/.openclaw/scripts/imsg-ssh chats --limit 1
```

### Step 5 — Server: Configure OpenClaw iMessage channel

In your OpenClaw config (usually `~/.openclaw/config.yaml` or `.json`):

```json
{
  "channels": {
    "imessage": {
      "enabled": true,
      "cliPath": "~/.openclaw/scripts/imsg-ssh",
      "remoteHost": "your-mac-username@your-mac-ip",
      "includeAttachments": true
    }
  }
}
```

### Step 6 — Start and pair

```bash
openclaw gateway start

# On first DM from a new contact, approve pairing:
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

That's it. Your server AI now receives and responds to iMessages through your Mac.

---

## Option B: Claude Code + iMessage MCP (Simple Outbound)

Best for: existing AI agents (Hermes, LangChain, custom) that need to send outbound texts without a full OpenClaw setup.

**What you get:** Outbound sends only. No inbound listening. Simple, minimal dependencies.

### Step 1 — Mac: Install Claude Code + iMessage MCP

```bash
npm install -g @anthropic-ai/claude-code
npm install -g imessage-mcp

export ANTHROPIC_API_KEY="sk-ant-..."
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc

# Grant Full Disk Access to Terminal in System Preferences → Privacy & Security
```

### Step 2 — Mac: Enable SSH + API key in environment

```bash
# Enable SSH
sudo systemsetup -setremotelogin on

# Make API key available to SSH sessions (SSH doesn't inherit shell env)
echo "ANTHROPIC_API_KEY=sk-ant-..." >> ~/.ssh/environment
sudo sh -c 'echo "PermitUserEnvironment yes" >> /etc/ssh/sshd_config'
sudo launchctl stop com.openssh.sshd && sudo launchctl start com.openssh.sshd
```

### Step 3 — Server: Configure SSH access

```bash
ssh-keygen -t ed25519 -C "ai-mac-bridge" -f ~/.ssh/mac_bridge
ssh-copy-id -i ~/.ssh/mac_bridge.pub USERNAME@MAC_IP

cat >> ~/.ssh/config << 'EOF'
Host mac-bridge
  HostName MAC_IP_OR_HOSTNAME
  User MAC_USERNAME
  IdentityFile ~/.ssh/mac_bridge
EOF
```

### One-liner test (after setup)

```bash
ssh mac-bridge "~/.local/bin/claude -p 'Use the imessage MCP to send a text to +1XXXXXXXXXX saying: bridge is live.' 2>&1"
```

### Python integration

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

## SSH Key Setup (Both Options)

```bash
# On your server — generate key
ssh-keygen -t ed25519 -C "server-ai" -f ~/.ssh/mac_bridge

# Copy to Mac
ssh-copy-id -i ~/.ssh/mac_bridge.pub USERNAME@MAC_IP

# Test
ssh -i ~/.ssh/mac_bridge USERNAME@MAC_IP "echo connected"
```

**Tip:** Use [DuckDNS](https://www.duckdns.org) (free) if your Mac's IP changes.

---

## Which Option Should I Use?

**Use OpenClaw if:**
- You want two-way conversations (AI responds to incoming texts)
- You need group chat, reactions, threaded replies
- You're building a full personal AI assistant
- You want OpenClaw's skill system, scheduling, memory, etc.

**Use Claude Code + iMessage MCP if:**
- You just need to send outbound texts from your existing agent
- You don't need inbound listening
- You want minimal setup with no gateway to manage

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `imsg: command not found` via SSH | Install with `brew install steipete/tap/imsg`. Use full path if needed. |
| `Full Disk Access` error | System Preferences → Privacy & Security → Full Disk Access → add Terminal.app |
| `claude: not found` via SSH | Use full path: `~/.local/bin/claude`. Find with `ssh mac-bridge "which claude"` |
| `ANTHROPIC_API_KEY` missing in SSH | Add to `~/.ssh/environment`, enable `PermitUserEnvironment yes` in sshd_config |
| `Permission denied (publickey)` | Re-run `ssh-copy-id`. Check `~/.ssh/authorized_keys` on Mac. |
| Mac IP keeps changing | Use DuckDNS for a stable hostname: https://www.duckdns.org |
| Attachments not working (OpenClaw) | Set `remoteHost` and `remoteAttachmentRoots` in OpenClaw config |
| OpenClaw pairing expired | Pairing requests expire after 1 hour — DM again to trigger a new one |

---

## Why Mac-Only?

iMessage is Apple-exclusive. The only way to programmatically send iMessages without jailbreaking is through Messages.app on macOS — which is what both `imsg` and the iMessage MCP do under the hood. There is no Linux/Windows equivalent.

**No Mac?** Use [Twilio](https://twilio.com) for SMS (no iMessage, but no Mac required).

---

## Security Notes

- Don't expose Mac SSH to the public internet — whitelist your server's IP in your router/firewall
- `~/.ssh/environment` stores API key in plaintext — `chmod 600 ~/.ssh/environment`
- Rotate SSH keys periodically
- Never commit API keys anywhere
- OpenClaw's pairing system prevents unknown contacts from triggering your AI without approval

---

## Contributing

PRs welcome. If you've got this working with a different AI framework, add an integration example under `examples/`.

---

## License

MIT
