---
title: "How to Set Up Your Own AI Agent with OpenClaw, Discord, and Claude"
date: 2026-04-04
slug: openclaw-discord-claude-setup
tags: ["openclaw", "ai", "discord", "tutorial", "claude"]
description: "A step-by-step guide to setting up a personal AI agent using OpenClaw, Discord as the interface, and Anthropic's Claude as the model."
---

# How to Set Up Your Own AI Agent with OpenClaw, Discord, and Claude

This is the practical guide. No philosophy, no "why" — just the steps to get a personal AI agent running on your machine, talking to you through Discord, powered by Claude.

By the end you'll have an agent you can message from your phone, your laptop, or anywhere Discord runs. It'll respond with Claude's intelligence and have access to tools you configure — shell, web search, file access, and more.

Total time: about 30 minutes.

## What You Need

- **A computer that stays on** — Mac, Linux, or Windows (WSL2 recommended). This runs the OpenClaw gateway.
- **Node.js 22.14+** — Node 24 recommended. Check with `node --version`.
- **An Anthropic API key** — Sign up at [console.anthropic.com](https://console.anthropic.com). You'll need a funded account.
- **A Discord account** — Plus a private server you control.

## Step 1: Install OpenClaw

One command:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

On Windows PowerShell:

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

This installs OpenClaw globally via npm and sets up the directory structure at `~/.openclaw/`.

## Step 2: Run Onboarding

```bash
openclaw onboard --install-daemon
```

The wizard walks you through:

1. **Model provider** — Select Anthropic
2. **API key** — Paste your Anthropic API key
3. **Model** — Choose `claude-sonnet-4-6` for a good balance of speed and capability (or `claude-opus-4-6` if you want the strongest model and don't mind the cost)
4. **Daemon** — The `--install-daemon` flag sets up OpenClaw to run in the background automatically

The wizard takes about 2 minutes. When it's done, verify the gateway is running:

```bash
openclaw gateway status
```

You should see it listening on port 18789.

## Step 3: Create a Discord Bot

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications)
2. Click **New Application** — name it whatever you want to call your agent
3. Click **Bot** in the sidebar
4. Set the **Username** to your agent's name
5. Under **Privileged Gateway Intents**, enable:
   - ✅ Message Content Intent (required)
   - ✅ Server Members Intent (recommended)
   - ✅ Presence Intent (optional)
6. Click **Reset Token** and copy the bot token — save it somewhere safe

## Step 4: Add the Bot to Your Server

If you don't have a private Discord server yet, create one (Create My Own → For me and my friends).

1. In the Developer Portal, click **OAuth2** in the sidebar
2. Under **OAuth2 URL Generator**, enable:
   - `bot`
   - `applications.commands`
3. Under **Bot Permissions**, enable:
   - View Channels
   - Send Messages
   - Read Message History
   - Embed Links
   - Attach Files
   - Add Reactions
4. Copy the generated URL at the bottom
5. Paste it in your browser, select your server, click **Continue**

Your bot should now appear in your server's member list (offline for now).

## Step 5: Get Your Discord IDs

Enable Developer Mode in Discord: User Settings → Advanced → Developer Mode.

Then right-click to copy:
- **Server ID** — right-click your server icon → Copy Server ID
- **Your User ID** — right-click your own avatar → Copy User ID

You'll need both in the next step.

## Step 6: Connect OpenClaw to Discord

Set the bot token securely (don't paste it in Discord chat):

```bash
export DISCORD_BOT_TOKEN="your-bot-token-here"
openclaw config set channels.discord.token \
  --ref-provider default \
  --ref-source env \
  --ref-id DISCORD_BOT_TOKEN
```

Then configure the Discord channel in your OpenClaw config (`~/.openclaw/openclaw.json`). Add the Discord section under `channels`:

```json
{
  "channels": {
    "discord": {
      "accounts": {
        "default": {
          "guilds": {
            "YOUR_SERVER_ID": {
              "channels": {
                "YOUR_CHANNEL_ID": {
                  "allow": true,
                  "requireMention": false
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Replace `YOUR_SERVER_ID` and `YOUR_CHANNEL_ID` with the IDs you copied. Set `requireMention` to `false` if you want the bot to respond to every message in that channel, or `true` if it should only respond when @mentioned.

Restart the gateway:

```bash
openclaw gateway restart
```

## Step 7: Send Your First Message

Go to the channel you configured in Discord and type something. Your agent should respond within a few seconds.

If it doesn't:
- Check `openclaw gateway status` — is it running?
- Check the logs: `openclaw gateway logs`
- Make sure the bot has permission to read and send in that channel
- Verify the channel ID is correct in your config

## Step 8: Enable DMs (Optional)

For pairing via DMs, right-click your server icon → Privacy Settings → enable **Direct Messages**. Then DM the bot directly — OpenClaw will walk you through pairing.

## What to Do Next

You've got a working agent. Here's where it gets interesting:

**Add more channels** — Create separate Discord channels for different projects and add them to the config. Each channel gets its own conversation context.

**Enable tools** — OpenClaw comes with built-in tools for shell execution, web search, web fetching, and browser control. Check the [tools documentation](https://docs.openclaw.ai/tools) to enable what you need.

**Write a SOUL.md** — Drop a `SOUL.md` file in `~/.openclaw/workspace/` to give your agent a personality, behavioral guidelines, and context about who you are. This file gets loaded at the start of every session.

**Set up cron jobs** — Schedule recurring tasks like daily briefings. See [my post on morning briefings](/posts/morning-briefing/) for how I use this.

**Build security guardrails** — If you're giving the agent real tool access, build a confirmation system. I wrote about [my approach to tiered authorization](/posts/ai-security-clearance-system/) and the [dual-channel token pattern](/posts/auth-token-pattern/) I use for high-risk actions.

## Cost

With Claude Sonnet, expect roughly $0.10-0.30 per conversation depending on length. Heavy daily use (briefings, email scanning, tool use) runs about $5-15/month. Opus is roughly 5x more expensive but noticeably more capable for complex reasoning tasks.

The OpenClaw software itself is free and open source.

---

*Cyberforks builds security workflows that don't suck. [cyberforks.com](https://cyberforks.com)*
