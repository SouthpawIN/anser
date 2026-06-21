# Profile Distributions: The .gitignore Pitfall

> Discovered June 2026 — user confusion about auto-exclusion vs git behavior

## The Confusion

The [Profile Distributions docs](https://hermes-agent.nousresearch.com/docs/user-guide/profile-distributions) say:

> "The git repo contains everything in the profile directory except things already excluded from distributions: `auth.json`, `.env`, `memories/`, `sessions/`, `state.db*`, `logs/`, `workspace/`, `*_cache/`, `local/`."

Users read this and assume `git add .` will skip those files automatically. It won't.

## Why

Hermes's exclusion list lives in `hermes_cli/profile_distribution.py` as a Python frozenset:

```python
USER_OWNED_EXCLUDE: frozenset = frozenset({
    "auth.json", ".env",
    "state.db", "state.db-shm", "state.db-wal",
    "hermes_state.db", "response_store.db",
    "memories", "sessions", "logs", "plans", "workspace", "home",
    "image_cache", "audio_cache", "document_cache",
    "browser_screenshots", "checkpoints", "sandboxes",
    "backups", "cache",
    "local",
    # ... plus more
})
```

This is used by Hermes CLI commands (`profile install`, `profile update`, `profile publish`) via `shutil.copytree(ignore=...)`. **Git knows nothing about it.** There is no auto-generated `.gitignore`.

## When `git add .` Works (and when it doesn't)

- **Fresh profile, never used:** Only contains `distribution.yaml`, `SOUL.md`, `config.yaml`, `skills/`, `cron/`. `git add .` is safe.
- **Used profile:** Has `memories/`, `sessions/`, `state.db`, `logs/`, etc. `git add .` will include ALL of them unless you have a `.gitignore`.

The docs' Step 3 shows `git init && git add .` assuming a fresh profile. If you've already used yours, this will leak sensitive data.

## The Fix

Always create a `.gitignore` before `git add`:

```gitignore
# Credentials & secrets
auth.json
.env

# Runtime databases & state
state.db
state.db-shm
state.db-wal
hermes_state.db
response_store.db
response_store.db-shm
response_store.db-wal
gateway.pid
gateway_state.json
processes.json

# User data
memories/
sessions/
logs/
plans/
workspace/
home/

# Caches & generated
image_cache/
audio_cache/
document_cache/
browser_screenshots/
cache/

# User customizations
local/

# Checkpoints & backups (can be huge)
checkpoints/
sandboxes/
backups/

# Infrastructure (shouldn't be in profile dir, but safe)
hermes-agent/
.worktrees/
profiles/
bin/
node_modules/
```

If files were already staged/tracked without `.gitignore`:

```bash
git rm -r --cached memories/ sessions/ logs/ state.db* .env auth.json
git add .gitignore
git commit -m "Add .gitignore, remove sensitive files from tracking"
```

## User-Facing Clarification Needed

The docs could be improved by:
1. Adding `.gitignore` creation as an explicit step before `git add .`
2. Clearly distinguishing "Hermes-internal exclusion" from "git exclusion"
3. Warning users who have already used their profile before publishing
