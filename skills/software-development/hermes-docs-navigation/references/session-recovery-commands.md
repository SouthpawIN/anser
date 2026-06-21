# Hermes Session Recovery Commands — /undo, /branch, /rewind, /rollback

> When the conversation is stuck (stale tool result, infinite loop, or you  
> want to explore a different approach without losing context).  
> **Verified: 2026-06-18 against Hermes source code and live docs.**

---

## Quick Decision Matrix

| Situation | Command | Creates New Session? | Needs Config? |
|-----------|---------|---------------------|----------------|
| Back up N bad turns (stale tool call) | `/undo N` | No (soft-deletes) | No |
| Fork entire convo for exploration | `/branch` | Yes (copy-all) | No |
| Roll back to filesystem checkpoint | `/rollback N` | No (restores files) | `checkpoints.enabled: true` |
| Rewind session DB to message ID | `/rewind <msg_id>` | No (mutates DB) | No (CLI/TUI only) |

---

## /undo [N] — Back Up N User Turns

**Use when:** A few recent turns went wrong (stale tool result, model went off
the rails). You want to discard the tail but keep everything before it.

```
/undo       # back up 1 turn
/undo 3     # back up 3 turns
```

**How it works:**
1. Call `SessionDB.list_recent_user_messages(session_id, limit=N)` to find
   the Nth user message back
2. Call `SessionDB.rewind_to_message(session_id, target_id)` — sets
   `active=0` on all rows after that point
3. Evict the cached AIAgent from memory
4. Echo the backed-up message text so the user can copy/edit and resend

**Source:** `gateway/slash_commands.py` → `_handle_undo_command()`,
`gateway/session.py` → `rewind_session()`, `hermes_state.py` →
`rewind_to_message()`.

**Key details:**
- Rows are **soft-deleted** (`active=0`), not destroyed
- Works on gateway (Discord, Telegram, etc.) and CLI
- No config required
- Rewind count is stored in session's `rewind_count` field

---

## /branch [name] — Fork Entire Conversation

**Use when:** You want to explore a different approach while keeping the
original session intact. NOT for trimming bad turns.

```
/branch                        # fork with auto-generated title
/branch "experiment-approach"  # fork with custom title
```

**How it works:**
1. Loads current session's full transcript
2. Creates a new session with `parent_session_id` linking to the original
3. Copies ALL messages to the new session
4. Sets `end_reason = 'branched'` on the original session
5. Switches the session store to the new session

**Source:** `gateway/slash_commands.py` → `_handle_branch_command()`.

**Key details:**
- Copies the ENTIRE history — no cutoff point selection
- Original session is preserved (resume it later)
- The `_branched_from` marker in `model_config` keeps branches visible in
  `/resume` and `/sessions`
- Branch children show up in session pickers (unlike compression children)

---

## /rollback [N] — Restore Filesystem Checkpoint

**Use when:** The agent made file changes you want to undo, and checkpoints
were enabled BEFORE the problem occurred.

```yaml
# Required in ~/.hermes/config.yaml:
checkpoints:
  enabled: true
```

```
/checkpoint list    # list available checkpoints
/rollback 3         # roll back to checkpoint #3
/rollback diff 3    # preview what would change
```

**Source:** `run_agent.py` → checkpoint management in `_checkpoint_mgr`.

**Key details:**
- Checkpoints are filesystem snapshots (not message-level)
- NOT retroactive — must be enabled before the problem
- A pre-rollback snapshot is saved automatically for safety

---

## /rewind — Session DB Truncation (CLI/TUI)

**Use when:** You need to truncate the session at a specific message ID
(advanced, usually from CLI/TUI).

The gateway also supports this through `session_store.rewind_session()` but
it's primarily surfaced as `/undo` for user-facing use.

**Source:** `cli.py` → `rewind_to_message()`, `tui_gateway/server.py` →
session rewind handler.

---

## Common Pitfall: When /undo Is Better Than /branch

Users often think `/branch` lets them pick a cutoff point. It doesn't.
`/branch` copies everything. If you want to drop the last 3 bad turns, use
`/undo 3` — it gives you exactly the cutoff behavior you want.
