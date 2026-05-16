# AGENTS.md — LLM Instructions for This Repo

> This file tells AI agents (Claude, Copilot, Cursor, etc.) exactly what this repo does and how to help users set it up.

---

## What This Repo Does

This repo teaches how to bridge a **Linux server AI** to a **Mac's iMessage** via SSH, giving the server AI the ability to send and receive texts from a real phone number.

**Stack:**
- Server: any Linux VPS/VM running an AI agent (Hermes, LangChain, custom, etc.)
- Mac: Claude Code CLI + iMessage MCP (the bridge)
- Protocol: SSH from server → Mac → Messages.app

---

## When a User Asks for Help Setting This Up

Walk them through these steps in order. Do not skip steps.

### Verify Prerequisites First

Ask or check:
1. Do they have a Mac? (Required — no Mac = no iMessage bridge. Suggest Twilio instead.)
2. Do they have a server (Linux VPS, cloud VM, etc.)?
3. Do they have an Anthropic API key?

If any are missing, stop and tell them what they need before continuing.

---

### Step-by-Step Setup

#### Mac Side

```bash
# 1. Install Claude Code
npm install -g @anthropic-ai/claude-code

# 2. Install iMessage MCP
npm install -g imessage-mcp

# 3. Set API key
export ANTHROPIC_API_KEY="sk-ant-..."
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc

# 4. Grant Full Disk Access to Terminal
# → System Preferences → Privacy & Security → Full Disk Access → add Terminal.app

# 5. Enable SSH on Mac
sudo systemsetup -setremotelogin on

# 6. Expose SSH environment to sessions
echo "ANTHROPIC_API_KEY=sk-ant-..." >> ~/.ssh/environment
sudo sh -c 'echo "PermitUserEnvironment yes" >> /etc/ssh/sshd_config'
sudo launchctl stop com.openssh.sshd && sudo launchctl start com.openssh.sshd
```

#### Server Side

```bash
# 1. Generate SSH key
ssh-keygen -t ed25519 -C "ai-mac-bridge" -f ~/.ssh/mac_bridge

# 2. Copy to Mac
ssh-copy-id -i ~/.ssh/mac_bridge.pub USERNAME@MAC_IP

# 3. Add to ~/.ssh/config
cat >> ~/.ssh/config << 'EOF'
Host mac-bridge
  HostName MAC_IP_OR_HOSTNAME
  User MAC_USERNAME
  IdentityFile ~/.ssh/mac_bridge
EOF

# 4. Test
ssh mac-bridge "~/.local/bin/claude -p 'Use the imessage MCP to send a text to +1XXXXXXXXXX saying: bridge is live' 2>&1"
```

---

### One-Liner Test (After Setup)

```bash
ssh mac-bridge "~/.local/bin/claude -p 'Use the imessage MCP to send a text to +1XXXXXXXXXX saying: hello from my server AI.' 2>&1"
```

If this sends a text, the bridge is working.

---

## Common Issues

| Symptom | Fix |
|---|---|
| `claude: not found` via SSH | Use full path. Find it with: `ssh mac-bridge "which claude"` |
| `ANTHROPIC_API_KEY` missing in SSH | Add to `~/.ssh/environment`, enable `PermitUserEnvironment yes` in sshd_config |
| iMessage MCP errors | Reinstall: `npm install -g imessage-mcp`. Check Full Disk Access. |
| `Permission denied (publickey)` | Check `~/.ssh/authorized_keys` on Mac. Run `ssh-copy-id` again. |
| Mac IP changes | Use DuckDNS (free): https://www.duckdns.org |
| Port 22 not reachable | Check Mac firewall. Consider port forwarding on router. |

---

## What to Never Do

- Do not commit API keys
- Do not expose Mac SSH to the public internet without IP whitelisting
- Do not run untrusted prompts through the bridge (it has iMessage access)

---

## File Structure

```
server-ai-phone-number/
├── README.md       ← Human-readable setup guide
├── AGENTS.md       ← This file (LLM instructions)
└── LICENSE
```

---

## Integration Patterns (Reference These When User Asks)

### Python one-liner

```python
import subprocess
def text(to, msg):
    subprocess.run(["ssh", "mac-bridge", f"~/.local/bin/claude -p 'Use imessage MCP to text {to}: {msg}' 2>&1"])
```

### Hermes Agent skill trigger

The send-text skill in Hermes calls:
```bash
ssh tyler-home "~/.local/bin/claude -p 'Use the imessage MCP tool to send a text to [NUMBER] saying: [MESSAGE]. Set confirm=true.' 2>&1"
```

---

## Tone for User Interactions

- Be direct. Users of this repo are technical.
- Don't suggest workarounds before trying the documented path first.
- If something fails, ask for the exact error output before guessing.
- If a user doesn't have a Mac, don't waste their time — tell them immediately and suggest Twilio.
