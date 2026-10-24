<h1 align="center">Server AI Phone Number</h1>

<p align="center">
  Give an AI agent on a Linux server the ability to send and receive iMessages, through a Mac you already own, over SSH.
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/github/license/tszaks/server-ai-phone-number" alt="MIT license"></a>
  <img src="https://img.shields.io/badge/bridge-macOS%20required-blue" alt="macOS required for the bridge">
  <img src="https://img.shields.io/badge/docs-only-lightgrey" alt="Documentation only, no code">
</p>

```text
Server AI (Linux)  --SSH-->  your Mac (Messages.app + imsg)  -->  iMessage / SMS
```

> **Read the [Security](#security) section before you set this up.** This bridge lets a program on a server read your Messages history and send texts as you.

Run your AI on a server and keep a Mac at home? This guide connects the two. The server AI reaches your Mac over SSH, and the Mac sends and receives through Messages.app with your own Apple ID and number.

This repository is a guide, not a program. It has step-by-step setup for two approaches, a decision tree to pick one, an [AGENTS.md](AGENTS.md) that coding agents can follow, and troubleshooting. It is for people who run self-hosted AI agents and want them to text.

## Features

- **Two supported paths.** OpenClaw with the `imsg` CLI for two-way iMessage, or Claude Code with an iMessage MCP server for simple outbound sends.

| Approach | Tool | Best for |
|---|---|---|
| **OpenClaw** (recommended) | `imsg` CLI + OpenClaw gateway | Full iMessage integration: receive, reply, react, groups |
| **Claude Code + iMessage MCP** | `claude` CLI + MCP | Simple outbound-only sends from any AI agent |

- **Provider agnostic on the OpenClaw path.** Use Anthropic, OpenAI, Gemini, Ollama or any backend OpenClaw supports.
- **Real number, real iMessage.** Messages go out from the Apple ID signed in on the Mac. There is no spoofed or virtual number.
- **Agent-readable.** [AGENTS.md](AGENTS.md) holds the same steps in a form an AI agent can follow, with a decision tree and a list of things not to do.

## How it works

```
+---------------------+        SSH        +---------------------+
|   Server AI         | ----------------> |   Your Mac          |
|   (OpenClaw,        |                   |   imsg CLI          |
|    Hermes, custom)  |                   |   Messages.app      |
+---------------------+                   +---------------------+
                                                     |
                                                     v
                                            iMessage / SMS
                                          (your real number)
```

Your server AI connects to your Mac via SSH. The Mac runs `imsg` (or Claude Code with the iMessage MCP) to send and receive messages through Messages.app, using your actual Apple ID and your real phone number.

## Security

**What access this grants.** Setting this up gives the server, and anything that controls the server, three powers on your Mac:

1. **An SSH login to your Mac user account.** Unless you restrict the key (see below), it can run any command that user can run.
2. **Read access to all your Messages.** `imsg` needs Full Disk Access to read `~/Library/Messages/chat.db`. That file holds every conversation and attachment record for the signed-in Apple ID, not only the ones the AI handles.
3. **The ability to send texts as you.** Automation access to Messages lets the bridge send iMessage and SMS from your real number to anyone, including people who have never texted the AI.

If the server is compromised, an attacker gets all three. If a stranger can text your number and your AI acts on what they write (Option A receives inbound messages), they can try to steer your AI with prompt injection. Treat inbound message text as untrusted input.

**Reduce the risk.**

- **Use a separate macOS user** for the bridge, signed in to Messages, instead of your main account. This limits what the SSH key can reach on disk. Consider a dedicated Apple ID and number if the AI does not need to be "you".
- **Restrict the SSH key** in the Mac's `~/.ssh/authorized_keys`. Standard OpenSSH options limit where it can log in from and what it can do:

  ```
  from="SERVER_IP",no-port-forwarding,no-agent-forwarding,no-X11-forwarding,no-pty ssh-ed25519 AAAA... ai-mac-bridge
  ```

  Test the bridge again after you add the options. For a tighter limit, use a forced `command=` that runs a small script allowing only `imsg`.
- **Do not expose Mac SSH to the public internet.** Allow only your server's IP in your router or firewall, or connect over a private network such as a VPN or Tailscale. Use key login only and turn off password login.
- **Keep API keys out of files you share.** Option B stores `ANTHROPIC_API_KEY` in plaintext in `~/.ssh/environment`. Run `chmod 600 ~/.ssh/environment`. That option also turns on `PermitUserEnvironment yes` in `sshd_config`, which loosens SSH on the whole Mac. If you can avoid Option B, do.
- **Gate who can talk to the AI.** On Option A, keep OpenClaw's pairing approval on, so an unknown contact cannot trigger the AI until you approve them. Approve only people you trust.
- **Never pass untrusted text to the send function.** The Python example runs an AI agent that has Messages access. Quote everything (the example uses `shlex.quote`).
- **Rotate keys** on a schedule, and never commit keys, tokens, phone numbers or hostnames to a repository.

**How to shut it off.** On the Mac, remove the server's line from `~/.ssh/authorized_keys`, run `sudo systemsetup -setremotelogin off` if you no longer need SSH, and remove Full Disk Access and Automation for `imsg` or the terminal in System Settings > Privacy & Security.

Report a problem with this guide through [SECURITY.md](SECURITY.md).

## Prerequisites

- A Mac signed into iMessage (this is the only hard requirement. No Mac, no bridge)
- A server running your AI (Linux VPS, cloud VM, etc.)
- SSH access from server to Mac
- An API key for whichever LLM provider you use (Anthropic, OpenAI, Gemini, Ollama, etc.)

---

## Option A: OpenClaw (recommended, full iMessage)

OpenClaw is an open source personal AI assistant with native iMessage support via `imsg`. When your gateway runs on Linux, you point it at a wrapper script that SSHes to your Mac and runs `imsg` there. This is OpenClaw's officially documented remote Mac path.

**What you get:** Inbound + outbound messages, reactions, threaded replies, group chats, attachments. Full two-way integration.

**Source:** [openclaw/openclaw](https://github.com/openclaw/openclaw) · [iMessage channel docs](https://docs.openclaw.ai/channels/imessage)

### Step 1: Mac: install imsg

```bash
brew install steipete/tap/imsg

# Verify
imsg --version
imsg status --json

# Grant permissions (run these once interactively to trigger macOS prompts)
# Full Disk Access + Automation required. Approve in System Preferences when prompted
imsg chats --limit 1
```

### Step 2: Mac: Enable SSH

```bash
sudo systemsetup -setremotelogin on
```

### Step 3: Server: Install OpenClaw + set your model

```bash
# Requires Node.js 20+
npx openclaw onboard
# Follow the interactive wizard. It walks through gateway, workspace, channel, and model setup
```

Or manual install:

```bash
npm install -g openclaw
openclaw --version
```

OpenClaw supports any major provider. Use whichever backend you already run:

```json
// ~/.openclaw/openclaw.json
{
  "agent": {
    "model": "<provider>/<model-id>"
  }
}
```

**Common examples:**

```json
{ "agent": { "model": "anthropic/claude-sonnet-4-5" } }
{ "agent": { "model": "openai/gpt-4o" } }
{ "agent": { "model": "google/gemini-2.0-flash" } }
{ "agent": { "model": "ollama/llama3.2" } }
```

Set the matching API key in your environment:

```bash
# Anthropic
export ANTHROPIC_API_KEY="sk-ant-..."

# OpenAI
export OPENAI_API_KEY="sk-..."

# Google Gemini
export GOOGLE_GENERATIVE_AI_API_KEY="..."

# Ollama (local, no key needed, just make sure Ollama is running)
# ollama serve
```

Full model config: https://docs.openclaw.ai/concepts/models

### Step 4: Server: create the SSH wrapper script

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

OpenClaw checks the Mac's SSH host key strictly. Connect once by hand (`ssh mac-bridge echo ok`) so the Mac's key is saved in `~/.ssh/known_hosts` on the server. In `remoteHost` below, use plain `user@host` with no spaces or SSH options. Put any other SSH options in `~/.ssh/config`.

Test the wrapper:

```bash
~/.openclaw/scripts/imsg-ssh chats --limit 1
```

### Step 5: Server: Configure OpenClaw iMessage channel

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

### Step 6: Start and pair

```bash
openclaw gateway start

# On first DM from a new contact, approve pairing:
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

That's it. Your server AI now receives and responds to iMessages through your Mac.

---

## Option B: Claude Code + iMessage MCP (simple outbound)

Best for: existing AI agents (Hermes, LangChain, custom) that need to send outbound texts without a full OpenClaw setup.

**Note:** This path uses Claude Code CLI specifically, so it requires an Anthropic API key. If you're on a different provider, use Option A (OpenClaw supports any model backend).

**What you get:** Outbound sends only. No inbound listening. Simple, minimal dependencies.

### Step 1: Mac: Install Claude Code + iMessage MCP

```bash
npm install -g @anthropic-ai/claude-code
npm install -g imessage-mcp

export ANTHROPIC_API_KEY="sk-ant-..."
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc

# Grant Full Disk Access to Terminal in System Preferences → Privacy & Security
```

### Step 2: Mac: Enable SSH + API key in environment

```bash
# Enable SSH
sudo systemsetup -setremotelogin on

# Make API key available to SSH sessions (SSH doesn't inherit shell env)
echo "ANTHROPIC_API_KEY=sk-ant-..." >> ~/.ssh/environment
sudo sh -c 'echo "PermitUserEnvironment yes" >> /etc/ssh/sshd_config'
sudo launchctl stop com.openssh.sshd && sudo launchctl start com.openssh.sshd
```

### Step 3: Server: Configure SSH access

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
import shlex
import subprocess

def send_imessage(recipient: str, message: str) -> str:
    prompt = f"Use the imessage MCP to send a text to {recipient} saying: {message}"
    # ssh runs this string in a shell on the Mac, so quote the prompt.
    remote = f"~/.local/bin/claude -p {shlex.quote(prompt)} 2>&1"
    result = subprocess.run(
        ["ssh", "mac-bridge", remote],
        capture_output=True, text=True, timeout=30
    )
    return result.stdout
```

Quoting matters. A message that contains a quote character would otherwise break out of the command and run on your Mac. Also see [Security](#security): this agent can send texts as you, so never pass text from an untrusted source straight into `message`.

---

## SSH Key Setup (Both Options)

```bash
# On your server, generate a key
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
- You're on OpenAI, Gemini, Ollama, or any non-Anthropic provider
- You're building a full personal AI assistant
- You want OpenClaw's skill system, scheduling, memory, etc.

**Use Claude Code + iMessage MCP if:**
- You just need to send outbound texts from your existing agent
- You don't need inbound listening
- You're already on Anthropic and want minimal setup with no gateway to manage

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
| Reads work but sends fail with `Not authorized to send Apple events to Messages. (-1743)` (OpenClaw over SSH) | The Automation grant went to `sshd-keygen-wrapper` instead of `imsg`. See the [OpenClaw iMessage setup docs](https://docs.openclaw.ai/channels/imessage/setup) and test a real send through the exact wrapper before you enable the channel. |
| OpenClaw cannot reach the Mac over SSH | OpenClaw checks the host key strictly. Run `ssh mac-bridge echo ok` once by hand so the key lands in `~/.ssh/known_hosts` on the server. |
| OpenClaw pairing expired | Pairing requests expire after 1 hour. DM again to trigger a new one |

---

## FAQ

### Why Mac only?

iMessage is Apple-exclusive. The only way to programmatically send iMessages without jailbreaking is through Messages.app on macOS, which is what both `imsg` and the iMessage MCP do under the hood. There is no Linux/Windows equivalent.

**No Mac?** Use [Twilio](https://twilio.com) for SMS (no iMessage, but no Mac required).
## Contributing

PRs welcome. If you have this working with a different AI framework, add an integration example under `examples/`. Please read [CONTRIBUTING.md](CONTRIBUTING.md) first. Security problems go through [SECURITY.md](SECURITY.md). Everyone taking part follows the [Code of Conduct](CODE_OF_CONDUCT.md).

Docs: [AGENTS.md](AGENTS.md) · [changelog](CHANGELOG.md)

## License

[MIT](LICENSE)
