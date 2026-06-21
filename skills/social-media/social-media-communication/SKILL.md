---
name: social-media-communication
description: "Social media & messaging: X/Twitter via xurl CLI, Yuanbao (元宝) groups with @mention support."
version: 1.0.0
author: Hermes Agent
license: MIT
tags: ["social-media", "twitter", "x", "xurl", "yuanbao", "messaging", "groups", "mentions"]
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [social-media, twitter, x, xurl, yuanbao, messaging, groups, mentions]
    category: social-media
    related_skills: [productivity-office-tools, smart-home-iot, creative-design-systems]
---

# Social Media & Communication

Unified skill for social media interaction and group messaging. Covers two platforms:

1. **X/Twitter** (`xurl`): Official X API CLI for posting, searching, DMs, media, v2 API access
2. **Yuanbao (元宝)** (`yuanbao`): Group interaction with @mentions, member queries, DMs

Load this skill when the user wants to:
- Post, search, or interact on X/Twitter
- Manage X media uploads (images, video)
- @mention users in Yuanbao groups
- Query Yuanbao group info and members
- Send direct messages in Yuanbao

---

## Quick Decision Guide

| Goal | Use This Section |
|------|------------------|
| Post/tweet on X | [X/Twitter (xurl)](#xtwitter-xurl) |
| Search X posts/users | [X/Twitter (xurl)](#xtwitter-xurl) |
| X DMs, likes, reposts, bookmarks | [X/Twitter (xurl)](#xtwitter-xurl) |
| Upload media to X | [X/Twitter (xurl)](#xtwitter-xurl) |
| @mention in Yuanbao group | [Yuanbao Groups](#yuanbao-groups-元宝) |
| Query Yuanbao group/members | [Yuanbao Groups](#yuanbao-groups-元宝) |
| Send Yuanbao DM | [Yuanbao Groups](#yuanbao-groups-元宝) |

---

## X/Twitter (xurl)

> **Source:** Absorbed from `xurl` skill. X/Twitter via xurl CLI: post, search, DM, media, v2 API.

### When to Use
- Posting, replying, quoting, deleting posts
- Searching posts and reading timelines/mentions
- Liking, reposting, bookmarking
- Following, unfollowing, blocking, muting
- Direct messages
- Media uploads (images and video)
- Raw access to any X API v2 endpoint
- Multi-app / multi-account workflows

### Critical Safety Rules
- **NEVER read, print, or send `~/.xurl` to LLM context**
- **NEVER ask user to paste credentials/tokens into chat**
- User must fill `~/.xurl` manually on their machine
- **NEVER use `--verbose` / `-v` in agent sessions** — leaks auth headers
- Forbidden flags: `--bearer-token`, `--consumer-key`, `--consumer-secret`, `--access-token`, `--token-secret`, `--client-id`, `--client-secret`
- Verify credentials only with: `xurl auth status`

### Installation
```bash
# Shell script (Linux + macOS)
curl -fsSL https://raw.githubusercontent.com/xdevplatform/xurl/main/install.sh | bash

# Homebrew (macOS)
brew install --cask xdevplatform/tap/xurl

# npm
npm install -g @xdevplatform/xurl

# Go
go install github.com/xdevplatform/xurl@latest
```

### One-Time User Setup (user runs outside agent)
1. Create app at https://developer.x.com/en/portal/dashboard
2. Set redirect URI: `http://localhost:8080/callback`
3. Copy Client ID and Client Secret
4. Register app locally:
   ```bash
   xurl auth apps add my-app --client-id YOUR_CLIENT_ID --client-secret YOUR_CLIENT_SECRET
   ```
5. Authenticate:
   ```bash
   xurl auth oauth2 --app my-app
   # If UsernameNotFound/403 on /2/users/me:
   xurl auth oauth2 --app my-app YOUR_USERNAME
   ```
6. Set default app:
   ```bash
   xurl auth default my-app
   ```
7. Verify:
   ```bash
   xurl auth status
   xurl whoami
   ```

### Quick Reference
| Action | Command |
|--------|---------|
| Post | `xurl post "Hello world!"` |
| Reply | `xurl reply POST_ID "Nice!"` |
| Quote | `xurl quote POST_ID "My take"` |
| Delete | `xurl delete POST_ID` |
| Read | `xurl read POST_ID` |
| Search | `xurl search "QUERY" -n 10` |
| Who am I | `xurl whoami` |
| User lookup | `xurl user @handle` |
| Timeline | `xurl timeline -n 20` |
| Mentions | `xurl mentions -n 10` |
| Like/Unlike | `xurl like POST_ID` / `xurl unlike POST_ID` |
| Repost/Undo | `xurl repost POST_ID` / `xurl unrepost POST_ID` |
| Bookmark/Remove | `xurl bookmark POST_ID` / `xurl unbookmark POST_ID` |
| Follow/Unfollow | `xurl follow @handle` / `xurl unfollow @handle` |
| Block/Unblock | `xurl block @handle` / `xurl unblock @handle` |
| Mute/Unmute | `xurl mute @handle` / `xurl unmute @handle` |
| Send DM | `xurl dm @handle "message"` |
| List DMs | `xurl dms -n 10` |
| Upload media | `xurl media upload path/to/file.mp4` |
| Media status | `xurl media status MEDIA_ID` |
| Raw API | `xurl /2/users/me` or `xurl -X POST /2/tweets -d '{"text":"..."}'` |

### Common Workflows

#### Post with Image
```bash
xurl media upload photo.jpg
xurl post "Check this out!" --media-id MEDIA_ID
```

#### Search and Engage
```bash
xurl search "golang" -n 10
xurl like POST_ID_FROM_RESULTS
xurl reply POST_ID_FROM_RESULTS "Great point!"
```

#### Multiple Apps
```bash
xurl auth default prod alice               # prod app, alice user
xurl --app staging /2/users/me             # one-off against staging
```

### Error Handling
| Symptom | Fix |
|---------|-----|
| Auth errors after OAuth | Token saved to `default` app → `xurl auth oauth2 --app my-app` then `xurl auth default my-app` |
| `unauthorized_client` | App type must be "Web app, automated app or bot" in X dashboard |
| `UsernameNotFound` after OAuth | `xurl auth oauth2 --app my-app YOUR_USERNAME` |
| 401 on every request | Check `xurl auth status` — verify `▸` points to app with oauth2 tokens |
| `client-forbidden` | Dashboard → Apps → Manage → Move to "Pay-per-use" → Production |
| `CreditsDepleted` | Buy credits (min $5) in Developer Console → Billing |
| Media processing failed | Add `--category tweet_image --media-type image/png` |

---

## Yuanbao Groups (元宝)

> **Source:** Absorbed from `yuanbao` skill. Yuanbao (元宝) groups: @mention users, query info/members.

### Critical: How Messaging Works
**Your text reply IS the message sent to the group/user.** The gateway automatically delivers your response text to the chat. You do NOT need any special "send message" tool — just reply normally.

When you include `@nickname` in your reply, the gateway converts it into a real @mention that notifies the user.

**NEVER say you cannot send messages or @mention users.** Just reply with the text you want sent.

### Available Tools
| Tool | When to Use |
|------|-------------|
| `yb_query_group_info` | Query group name, owner, member count |
| `yb_query_group_members` | Find user, list bots, list all members, get nickname for @mention |
| `yb_send_dm` | Send private/direct message (DM/私信) with optional media |

### @Mention Workflow
When you need to @mention / 艾特 someone:

1. Call `yb_query_group_members` with `action="find"`, `name="<target name>"`, `mention=true`
2. Get the exact nickname from the response
3. Include `@nickname` in your reply text — the gateway handles the rest

**Example:** User says "帮我艾特元宝"

Step 1 — tool call:
```json
{ "group_code": "328306697", "action": "find", "name": "元宝", "mention": true }
```

Step 2 — your reply (this gets sent with working @mention):
```
@元宝 你好，有人找你！
```

**Rules:**
- Call `yb_query_group_members` first to get exact nickname — do NOT guess
- @mention format: `@nickname` with a space before the @ sign
- Your reply text IS the message — it WILL be sent and the @mention WILL work
- Be concise. Do NOT explain how @mention works to the user.

### Send DM (Private Message) Workflow
When someone asks to send a private message / 私信 / DM:

1. Call `yb_send_dm` with `group_code`, `name` (target user), and `message`
2. Tool automatically finds user and sends DM
3. Report result to user

**Example:** User says "给 @用户aea3 私信发一个 hello"
```json
yb_send_dm({ "group_code": "535168412", "name": "用户aea3", "message": "hello" })
```

**Example with media:**
```json
yb_send_dm({
  "group_code": "535168412",
  "name": "用户aea3",
  "message": "Here is the image",
  "media_files": [{"path": "/tmp/photo.jpg"}]
})
```

**Rules:**
- Extract `group_code` from chat_id: `group:535168412` → `535168412`
- If you know `user_id`, pass it directly to skip lookup
- If multiple users match, tool returns candidates — ask user to clarify
- Do NOT use `send_message` tool for Yuanbao DMs — use `yb_send_dm`
- Supports media: images (.jpg/.png/.gif/.webp/.bmp) as images, other files as documents

### Query Group Info
```json
yb_query_group_info({ "group_code": "328306697" })
```

### List Members
| Action | Description |
|--------|-------------|
| `find` | Search by name (partial match, case-insensitive) |
| `list_bots` | List bots and Yuanbao AI assistants |
| `list_all` | List all members |

### Notes
- `group_code` from chat_id: `group:328306697` → `328306697`
- Groups called "派 (Pai)" in Yuanbao app
- Member roles: `user`, `yuanbao_ai`, `bot`

---

## Decision Matrix

| Task | Platform | Tool |
|------|----------|------|
| Post on X/Twitter | X | `xurl post` |
| Search X posts | X | `xurl search` |
| X DMs | X | `xurl dm` / `xurl dms` |
| Upload to X | X | `xurl media upload` |
| @mention in group | Yuanbao | `yb_query_group_members` + `@nickname` in reply |
| Query group members | Yuanbao | `yb_query_group_members` |
| Send Yuanbao DM | Yuanbao | `yb_send_dm` |

---

## Related Skills

- **`productivity-office-tools`** — Email (Himalaya), Teams meetings, calendar
- **`smart-home-iot`** — Notifications via smart home
- **`research-knowledge-retrieval`** — Monitor social media for research
