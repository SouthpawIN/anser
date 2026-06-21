# Stale Tool Result Recovery — Branching & Rewinding

## Symptom

Hermes hits a tool result that can't be resolved to a matching tool request ID.
The conversation loop keeps trying to process it and fails — the agent is stuck
in a loop trying to handle an orphaned tool response.

## Recovery Mechanisms (No Config Needed)

### `/branch` (Recommended — Creates a Fork)

Creates a new session that inherits all context up to a chosen point, then
continues fresh from there. The original session is preserved.

```
/branch          # Opens picker to choose branch point
/branch 5        # Branches from 5 messages back
```

**How it works:** New session gets `parent_session_id` → original, with
`end_reason = 'branched'`. All context up to branch point is injected into
the new session.

### `/rewind` (Mutates Current Session)

Truncates the session's message list in the database back to a specific
message, then continues in the same session.

```
/rewind <message_number>
```

Uses `rewind_to_message()` in the session DB. Increments `rewind_count`.

## Checkpoints + `/rollback` (Requires Config)

```yaml
# ~/.hermes/config.yaml
checkpoints:
  enabled: true
```

Then:
```
/checkpoint list     # See available checkpoints
/rollback 3          # Roll back to checkpoint #3
```

Only works if checkpoints were enabled BEFORE the problem — not retroactive.

## Quick Reference

| Command | New Session? | Preserves Context? | Config Needed? |
|---------|-------------|--------------------|----------------|
| `/branch` | Yes (fork) | Up to branch point | No |
| `/rewind` | No (mutates) | Up to rewind point | No |
| `/rollback` | No (mutates) | To checkpoint | `checkpoints.enabled: true` |

## Likely Bug Causes

1. **API transport desync** — tool result arrives but tool_call_id doesn't match
   any pending request (orphaned from a cleaned-up previous turn)
2. **Compression artifact** — context compression drops a tool request but
   leaves its orphaned result
3. **Race condition** — two tool calls interleaved, one result arrives out of order

## For Bug Reports

Collect: session ID, `~/.hermes/logs/errors.log`, model/provider being used.
