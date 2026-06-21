# Discord Error 30032 — Maximum Application Commands Reached (100)

## Symptom

Gateway crashes during startup with:

```
discord.errors.HTTPException: 400 Bad Request (error code: 30032): 
Maximum number of application commands reached (100).
```

The traceback points to `_safe_sync_slash_commands` → `upsert_global_command` in
`plugins/platforms/discord/adapter.py`.

## Root Cause

Discord enforces a hard cap of 100 global application commands per bot. Hermes
maps skills, tools, and built-in commands to Discord slash commands. The
`"safe"` sync policy (default) does incremental upserts without removing stale
commands, so you silently accumulate past the 100 limit. The 30032 error is
NOT caught as a rate-limit (it's a 400, not a 429), so it bypasses the retry
logic and crashes the entire gateway.

## Fixes (Pick One)

### Fix 1: Switch to `bulk` Sync Policy (Recommended)

Uses Discord's `tree.sync()` which does a full tree-replace — deletes old
commands and creates new ones in one operation.

```bash
# In ~/.hermes/.env:
DISCORD_COMMAND_SYNC_POLICY=bulk
```

Then restart the gateway.

### Fix 2: Disable Slash Command Sync

```bash
DISCORD_COMMAND_SYNC_POLICY=off
```

Prevents Hermes from touching Discord's command registry. In-chat prefix
commands (e.g., `/help`, `/model`) still work — Hermes intercepts these as
text-prefix commands even without registered slash commands.

### Fix 3: Clear Commands via Discord API

Nuclear option — delete all global commands, then let Hermes re-sync a
clean set:

```bash
curl -X PUT -H "Authorization: Bot <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '[]' \
  https://discord.com/api/v10/applications/<APP_ID>/commands
```

Then restart Hermes with `DISCORD_COMMAND_SYNC_POLICY=bulk`.

### Fix 4: Reduce Hermes Skill Count

Disable skills you don't need on Discord:

```yaml
# ~/.hermes/config.yaml
skills:
  platform_denylist:
    discord:
      - some-heavy-skill
      - another-skill
```

## Valid Policy Values

From `adapter.py`:
```python
_DISCORD_COMMAND_SYNC_POLICIES = {"safe", "bulk", "off"}
```

- `safe` (default): fingerprint-based incremental sync
- `bulk`: full tree-replace via `tree.sync()`
- `off`: no slash command registration
