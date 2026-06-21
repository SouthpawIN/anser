# Unified Profile Management Architecture

> Researched: 2026-06-11 · Hermes Agent v0.16.0+

## Overview

Hermes Agent profiles are isolated environments under `~/.hermes/profiles/<name>/`. The "default" profile lives at `~/.hermes/` (no subdirectory). Each profile has its own config, skills, plugins, cron, memories, sessions, and logs.

## Key Architecture Insights

### Desktop Profile Switcher (`apps/desktop/src/store/profile.ts`)

The desktop app implements a **workspace switcher model** (Slack/VS Code/Linear style):

- **Primary socket**: Active profile gets a live gateway WebSocket
- **Secondary sockets**: Background profiles with running sessions keep their own sockets alive concurrently
- **Lazy swap**: `ensureGatewayForProfile()` opens the target's socket without closing the source
- **Unified session list**: "All profiles" mode fans every profile's sessions into one grouped list
- **Hotkey navigation**: `⌘1` for default, `⌘2+` for named profiles

Key stores:
- `$activeGatewayProfile` — which profile the live WebSocket is connected to
- `$profileScope` — sidebar context (concrete profile or `ALL_PROFILES`)
- `$newChatProfile` — where new chats open
- `$freshSessionRequest` — bumped on profile switch to request a fresh session

### Desktop Gateway Multiplexing (`apps/desktop/src/store/gateway.ts`)

Manages multiple gateway sockets:
- Primary (window backend) + secondaries (pooled per profile)
- `ensureGatewayForProfile(profile)` — lazily opens socket for target profile
- `pruneSecondaryGateways(keep)` — closes sockets for inactive profiles
- `reconnectSecondaryGateways()` — wake signal for sleep/network recovery

### Web Dashboard (`web/src/pages/ProfilesPage.tsx`)

Full card-based profile management UI with:
- Inline editors: model, description, SOUL (system prompt)
- One-click "set active" per profile
- Gateway status display
- Create/Rename/Delete lifecycle
- Clone with state option

### Unified Gateway Routing

Commit `02d6bf1c3`: "full multi-profile support over one global-remote dashboard"

One gateway backend serves multiple profiles via `?profile=<name>` query parameter:
- Reads each profile's `state.db` off the remote host's disk
- Sessions API: `/api/profiles/sessions?profile=all` aggregates all profiles
- Desktop interceptor routes per-session reads with `?profile=` preserved

### CLI Profile Commands

```bash
hermes profile list              # All profiles (◆ = active)
hermes profile create <name>     # Create with options
hermes profile use <name>        # Set as default
hermes profile show <name>       # Details
hermes profile describe <name>   # Read/set description
hermes profile rename <old> <new>
hermes profile delete <name>
hermes profile alias <name>      # Shell alias
hermes profile export/import     # tar.gz portability
hermes profile install <url>     # From git URL
hermes profile update <name>     # Re-pull distribution
hermes profile info <name>       # Distribution metadata
```

## Profile Storage Layout

```
~/.hermes/                          # Default profile
  config.yaml, .env, skills/, plugins/, cron/, memories/, state.db, logs/

~/.hermes/profiles/<name>/          # Named profiles
  config.yaml, .env, skills/, plugins/, cron/, memories/, state.db, logs/
  audio_cache/, SOUL.md
```

## Troubleshooting

| Symptom | Check |
|---------|-------|
| Dashboard dropdown not showing | `hermes update` (v0.16.0+), clear browser cache, verify >1 profile |
| Wrong profile's settings | Check which profile is selected in top-left dropdown |
| Gateway not running | `hermes profile list` shows gateway status; start with `hermes gateway --profile <name>` |
| Can't delete profile | Stop gateway first: `hermes gateway stop --profile <name>` |
| Sessions not showing in desktop | Check Settings → Gateway; in global remote mode, verify `?profile=` routing |
