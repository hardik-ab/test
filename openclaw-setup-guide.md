# OpenClaw Setup Guide — Secure & Scalable

OpenClaw is a self-hosted, local-first personal AI assistant that connects to 20+ messaging platforms (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, and more). This guide covers installation, security hardening, and scalable deployment.

---

## Prerequisites

| Requirement | Minimum | Recommended |
|---|---|---|
| Node.js | 22.14+ (LTS) | 24.x |
| OS | macOS, Linux, Windows (WSL2) | Linux (Ubuntu 22.04+) |
| AI Provider Key | Anthropic, OpenAI, or Google | Anthropic (Claude) |
| RAM | 512 MB | 1 GB+ |

---

## 1. Installation

### Option A — Installer Script (Recommended for first-time setup)

The installer auto-detects your OS and Node version:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

> Always inspect installer scripts before piping to bash:
> `curl -fsSL https://openclaw.ai/install.sh | less`

### Option B — npm / pnpm

```bash
npm install -g openclaw@latest
# or
pnpm add -g openclaw@latest
```

### Option C — Build from Source

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install && pnpm build
```

---

## 2. Onboarding

Run the interactive onboarding wizard. It configures the Gateway, model auth, workspace, and channels in one session:

```bash
openclaw onboard --install-daemon
```

This registers the background daemon that keeps the Gateway running persistently. Budget **15–30 minutes** depending on whether you need to set up API keys first.

---

## 3. Minimal Configuration

At minimum, specify your AI model in your OpenClaw config:

```json
{
  "agent": {
    "model": "anthropic/claude-sonnet-4-20250514"
  }
}
```

### API Keys — Never Hardcode

Store API keys in environment variables:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENCLAW_API_KEY="..."
```

Or use a `.env` file (never commit it):

```
# .env
ANTHROPIC_API_KEY=sk-ant-...
```

Add `.env` to `.gitignore`:

```
echo ".env" >> .gitignore
```

---

## 4. Security Hardening

### 4.1 DM Pairing (Built-in Trust Model)

By default, OpenClaw treats **inbound DMs as untrusted input**. Unknown senders receive a code they must return to you for approval before their messages are processed. This is enabled by default — do not disable it.

### 4.2 Sandbox Non-Main Sessions

For team or multi-user deployments, sandbox each non-main session in its own Docker container:

```bash
docker run --rm -it \
  --name openclaw-session-user1 \
  -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
  --network=internal \
  openclaw/openclaw:latest
```

This limits blast radius if any session is compromised.

### 4.3 Restrict Agent Tool Access

In your config, explicitly allowlist only the tools each agent session needs:

```json
{
  "agent": {
    "model": "anthropic/claude-sonnet-4-20250514",
    "tools": ["read_file", "web_search"]
  }
}
```

Do not grant file-write or shell-exec tools unless strictly required.

### 4.4 Never Expose the Gateway Port Publicly

The Gateway runs locally at `ws://127.0.0.1:18789` by default. **Do not bind this to 0.0.0.0 or expose it to the internet directly.** Use Tailscale (see Section 5) for remote access.

### 4.5 Run Under a Dedicated Low-Privilege User

```bash
# Create a dedicated user
sudo useradd -m -s /bin/bash openclaw

# Run OpenClaw as that user
sudo -u openclaw openclaw onboard --install-daemon
```

### 4.6 Keep OpenClaw Updated

```bash
npm install -g openclaw@latest
```

Subscribe to the **stable** channel unless you need new features:

```bash
openclaw channel stable
```

---

## 5. Scalable Remote Deployment

### 5.1 Run the Gateway on a Remote Linux Instance

For always-on availability, run OpenClaw on a VPS (e.g., DigitalOcean, AWS Lightsail, Hetzner):

```bash
# On the remote server
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

### 5.2 Secure Remote Access via Tailscale

Use [Tailscale](https://tailscale.com) to create a private mesh network — no open ports needed:

```bash
# Install Tailscale on server
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Expose OpenClaw gateway over Tailscale
openclaw serve --tailscale
```

For public-facing webhooks (e.g., WhatsApp callbacks), use **Tailscale Funnel**:

```bash
tailscale funnel 18789
```

This gives you an HTTPS endpoint backed by Tailscale — no nginx or Let's Encrypt config needed.

### 5.3 Docker Compose (Multi-Session Deployment)

```yaml
# docker-compose.yml
version: '3.9'
services:
  openclaw-gateway:
    image: openclaw/openclaw:latest
    restart: unless-stopped
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    volumes:
      - ./config:/home/openclaw/.openclaw
    networks:
      - openclaw-net
    ports:
      - "127.0.0.1:18789:18789"   # Bind to localhost only

  openclaw-session-a:
    image: openclaw/openclaw:latest
    restart: unless-stopped
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    networks:
      - openclaw-net

networks:
  openclaw-net:
    driver: bridge
    internal: true   # No direct internet access for containers
```

Start with:

```bash
docker compose up -d
```

### 5.4 Health Monitoring

OpenClaw ships with a built-in diagnostics tool:

```bash
openclaw doctor
```

This checks daemon health, config validity, channel connectivity, and API key status. Integrate it into your monitoring pipeline or a cron job:

```bash
# crontab -e
*/15 * * * * openclaw doctor --json >> /var/log/openclaw-health.log 2>&1
```

---

## 6. Architecture Overview

```
                    ┌─────────────────────────────┐
                    │         OpenClaw            │
                    │                             │
  Channels ──────▶  │  Gateway (WebSocket)        │
  (WhatsApp,        │  ws://127.0.0.1:18789       │
   Telegram,        │           │                 │
   Slack, etc.)     │  Pi Agent Runtime (RPC)     │
                    │           │                 │
                    │  AI Model (Anthropic, etc.) │
                    └─────────────────────────────┘
                                │
              ┌─────────────────┴──────────────────┐
              │                                    │
       Tailscale / SSH                       Docker Sandbox
       (Remote access)                      (Per-session isolation)
```

**Gateway** — Central control plane. Manages sessions, channel routing, tool calls, and event dispatch.

**Pi Agent Runtime** — The RPC layer that executes tool calls and communicates with the AI provider.

**Multi-channel inbox** — Normalises inbound messages from 20+ platforms into a single queue.

---

## 7. Quick Reference

| Task | Command |
|---|---|
| Install | `npm install -g openclaw@latest` |
| Onboard + start daemon | `openclaw onboard --install-daemon` |
| Check health | `openclaw doctor` |
| Switch to stable channel | `openclaw channel stable` |
| Update | `npm install -g openclaw@latest` |
| Serve via Tailscale | `openclaw serve --tailscale` |

---

## 8. Security Checklist

- [ ] API keys stored in env vars or `.env`, never hardcoded or committed
- [ ] `.env` in `.gitignore`
- [ ] DM pairing enabled (default — do not disable)
- [ ] Gateway port bound to `127.0.0.1` only, not `0.0.0.0`
- [ ] Remote access via Tailscale, not open ports
- [ ] Non-main sessions sandboxed in Docker containers
- [ ] Agent tool access explicitly allowlisted
- [ ] Running under a dedicated low-privilege OS user
- [ ] Auto-updates enabled or regular manual updates scheduled
- [ ] `openclaw doctor` integrated into monitoring

---

## Sources

- [OpenClaw Official Docs — Install](https://docs.openclaw.ai/install)
- [OpenClaw GitHub Repository](https://github.com/openclaw/openclaw)
- [How to Run OpenClaw with DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-openclaw)
- [OpenClaw Setup Tutorial 2026](https://www.shareuhack.com/en/posts/openclaw-setup-tutorial-2026)
- [OpenClaw Setup Guide — Verdent](https://www.verdent.ai/guides/openclaw-setup-guide-from-zero-to-ai-assistant)
