# OpenClaw Setup Guide

OpenClaw is a self-hosted AI assistant that connects to your messaging apps (WhatsApp, Telegram, Slack, Discord, Signal, and 15+ more). You run it yourself — your data stays on your machine.

---

## Should you use a separate device?

**Yes, if you can.** Here's why in plain terms:

- OpenClaw runs a background service all day and night. On your main laptop, this drains battery and uses up memory while you're working.
- If something breaks, it won't affect your primary machine.
- A dedicated device (old laptop, Raspberry Pi 4, cheap cloud VPS) stays always-on so your assistant is always reachable.

**If you only have one laptop** — it still works fine there. Just expect some extra RAM usage in the background.

**Best device options (cheapest to easiest):**

| Device | Cost | Notes |
|---|---|---|
| Spare/old laptop | $0 | Best if you have one sitting around |
| Raspberry Pi 4 | ~$60 one-time | Tiny, quiet, always-on |
| Cloud VPS (Hetzner, DigitalOcean) | ~$5–6/month | No hardware to manage |

---

## Step-by-Step Setup

### Step 1 — Check Node.js

OpenClaw needs Node.js version 22 or newer. Check what you have:

```bash
node --version
```

If it shows `v22.x` or higher, you're good. If not, install it from [nodejs.org](https://nodejs.org).

---

### Step 2 — Install OpenClaw

```bash
npm install -g openclaw@latest
```

Then run the setup wizard — it walks you through everything step by step:

```bash
openclaw onboard --install-daemon
```

This will ask you for:
- Your AI provider API key (e.g. Anthropic)
- Which messaging platforms to connect
- Whether to start on boot

Budget **15–30 minutes** for your first run.

---

### Step 3 — Store your API key safely

Never type your API key directly into a config file or share it in chat. Instead, create a `.env` file:

```
# .env
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

Then make sure it's never accidentally uploaded to GitHub:

```bash
echo ".env" >> .gitignore
```

---

### Step 4 — Access it remotely (without opening ports)

If OpenClaw runs on a separate device, you need a safe way to reach it from your main laptop or phone.

**Use Tailscale** — it's free, takes 5 minutes to set up, and creates a private encrypted tunnel between your devices. No open firewall ports needed.

```bash
# Install Tailscale on the OpenClaw device
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Tell OpenClaw to serve over Tailscale
openclaw serve --tailscale
```

Now you can reach OpenClaw from any of your devices on your Tailscale network.

---

### Step 5 — Check everything is working

```bash
openclaw doctor
```

This is the built-in health check. Run it any time something feels off — it tells you exactly what's misconfigured.

---

## Security — The Simple Rules

You don't need to be a security expert. Just follow these:

1. **Don't share your API key** — keep it in `.env`, not in your config or code
2. **Leave DM pairing ON** (it's on by default) — strangers can't just message your assistant; they have to be approved first
3. **Don't expose port 18789 to the internet** — use Tailscale instead
4. **Keep it updated** — run `npm install -g openclaw@latest` every few weeks
5. **Run `openclaw doctor` regularly** — catches problems before they become issues

---

## Quick Reference

| What you want to do | Command |
|---|---|
| Install | `npm install -g openclaw@latest` |
| First-time setup | `openclaw onboard --install-daemon` |
| Check health | `openclaw doctor` |
| Update | `npm install -g openclaw@latest` |
| Remote access via Tailscale | `openclaw serve --tailscale` |

---

## Security Checklist

- [ ] API keys in `.env`, not hardcoded anywhere
- [ ] `.env` added to `.gitignore`
- [ ] DM pairing left ON (default)
- [ ] Not exposing port 18789 publicly
- [ ] Using Tailscale for remote access
- [ ] Running on a separate/dedicated device if possible
- [ ] `openclaw doctor` run and all checks passing

---

## Sources

- [OpenClaw Official Docs](https://docs.openclaw.ai/install)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [Tailscale](https://tailscale.com)
