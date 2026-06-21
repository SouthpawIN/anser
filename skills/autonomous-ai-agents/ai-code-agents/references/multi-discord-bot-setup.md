# Multi-Discord-Bot Setup with Hermes Profiles

> Complete tutorial: two independent Discord bots managed from one dashboard
> Hermes Agent v0.16.0+

## Architecture

```
┌─────────────────────────────────────────┐
│         ONE Web Dashboard (:9119)        │
│  ┌─────────────────────────────────┐    │
│  │ Profile dropdown [discord-mod ▼]│    │
│  │ - Manage config, skills, env    │    │
│  │ - Switch to discord-helper      │    │
│  │ - View sessions for either      │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
         │                    │
    manages both        manages both
         │                    │
    ┌────▼─────┐      ┌──────▼──────┐
    │ Gateway  │      │  Gateway    │
    │ mod PID  │      │  helper PID │
    └────┬─────┘      └──────┬──────┘
         │                    │
    ┌────▼─────┐      ┌──────▼──────┐
    │ Discord  │      │  Discord    │
    │ Bot #1   │      │  Bot #2     │
    │ "ModBot" │      │ "HelperBot" │
    └──────────┘      └─────────────┘
```

Each profile is fully isolated:
```
~/.hermes/profiles/discord-mod/
  .env           ← DISCORD_BOT_TOKEN=***.yaml    ← Discord config + model
  skills/        ← Moderation skills
  state.db       ← Own session history

~/.hermes/profiles/discord-helper/
  .env           ← DISCORD_BOT_TOKEN=***.yaml    ← Different model/personality
  skills/        ← Helper skills
  state.db       ← Own session history
```

## Step 1: Create Discord Applications

Go to https://discord.com/developers/applications — create two applications:
- **Bot #1** (ModBot): Reset Token, enable MESSAGE CONTENT INTENT + SERVER MEMBERS INTENT, OAuth2: bot scope + mod permissions
- **Bot #2** (HelperBot): Same but minimal permissions (Send Messages, Read Message History)

## Step 2: Create Hermes Profiles

```bash
hermes profile create discord-mod \
  --description "Moderation bot — enforces rules, manages roles, handles reports" \
  --model openrouter/qwen/qwen3.6-flash

hermes profile create discord-helper \
  --description "Community helper — answers FAQs, welcomes new members, shares resources" \
  --model openrouter/anthropic/claude-sonnet-4
```

## Step 3: Set Discord Bot Tokens

```bash
# Moderation bot
mkdir -p ~/.hermes/profiles/discord-mod
echo 'DISCORD_BOT_TOKEN=YOUR_MOD_TOKEN_HERE' >> ~/.hermes/profiles/discord-mod/.env

# Helper bot
mkdir -p ~/.hermes/profiles/discord-helper
echo 'DISCORD_BOT_TOKEN=YOUR_HELPER_TOKEN_HERE' >> ~/.hermes/profiles/discord-helper/.env
```

Alternative: use the dashboard → Environment page (select profile from dropdown first).

## Step 4: Configure per Profile

Edit `~/.hermes/profiles/discord-mod/config.yaml`:
```yaml
discord:
  require_mention: true
  free_response_channels: ''
agent:
  personality: |
    You are ModBot, a strict-but-fair Discord server moderator...
disabled_toolsets:
  - image_gen
  - browser
  - tts
```

Edit `~/.hermes/profiles/discord-helper/config.yaml`:
```yaml
discord:
  require_mention: true
agent:
  personality: |
    You are HelperBot, a friendly and knowledgeable community assistant...
disabled_toolsets: []
```

## Step 5: Install and Start Gateways

```bash
# Install both
hermes profile use discord-mod && hermes gateway install
hermes profile use discord-helper && hermes gateway install

# Start both
hermes profile use discord-mod && hermes gateway start
hermes profile use discord-helper && hermes gateway start

# Verify
hermes gateway list
```

## Step 6: Manage Both from One Dashboard

```bash
hermes dashboard
```

- **Top-left dropdown** selects which profile you're administering
- Switch to edit config, env, model, skills, cron, sessions, logs for either bot
- **No second dashboard needed** — that's the whole point of unified profile management

## Step 7: Test

```
@ModBot !warn @troublemaker spamming general chat
@HelperBot what's the policy on self-promotion?
```

## Quick Commands Reference

```bash
hermes profile create <name> --model <provider/model> --description "..."
hermes profile use <name>
hermes gateway install / start / stop
hermes gateway list
hermes gateway stop --all
hermes dashboard
hermes profile use <name> && hermes logs --follow
```
