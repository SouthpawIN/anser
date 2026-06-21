# Telegram sendMessageDraft Streaming Flicker

> Recurring pitfall — messages appear then disappear after disabling topics mode.  
> GitHub: [#26157](https://github.com/NousResearch/hermes-agent/issues/26157)  
> Verified: June 12, 2026 — fix is `streaming.transport: edit`

## Symptoms

- Agent responses appear briefly in Telegram DM, then partially or fully disappear
- Text flickers: appears → collapses → reappears → collapses (cycling)
- User messages and system notifications are unaffected (they don't stream)
- Commonly triggered **after disabling topics mode** (`/topic off` or removing `dm_topics` from config)

## Root Cause

Hermes defaults to `streaming.transport: auto`. On Telegram DMs with `python-telegram-bot` ≥ 22.6, this enables `sendMessageDraft` — Telegram's native draft-streaming API.

`sendMessageDraft` frames are typing-preview animations, not real messages. Each new draft frame with more accumulated text causes the Telegram client to **re-render the preview from scratch**, collapsing existing content briefly before showing the updated text.

When topics are enabled, the conversation lives inside a thread where `sendMessageDraft` doesn't work, so Hermes falls back to the stable `editMessageText` path. Disabling topics drops you back to regular DM where draft streaming activates.

## Fix

```yaml
# ~/.hermes/config.yaml
streaming:
  transport: edit    # Force progressive editMessageText (no flicker)
```

Then `hermes gateway restart`.

**Alternative:** `streaming.enabled: false` disables streaming entirely (one-shot message, guaranteed no flicker).

## Why the fix works

| Transport | Mechanism | Flicker? | Works in topics? |
|-----------|----------|----------|-----------------|
| `edit` (default after fix) | Progressive `editMessageText` | No | Yes |
| `auto` (old default) | `sendMessageDraft` when available, else `edit` | Yes (DMs) | Falls back to `edit` in threads |
| `off` | Single `sendMessage` | No | Yes |

## Related Issues

- [#16668](https://github.com/NousResearch/hermes-agent/issues/16668) — Streaming flood control leaves partial message + duplicate final
- [#42765](https://github.com/NousResearch/hermes-agent/issues/42765) — Topic streamed reply stops after first overflow chunk
