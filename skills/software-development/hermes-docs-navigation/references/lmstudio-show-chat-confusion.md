# LM Studio "Show Chat" vs Hermes — Common User Confusion

## Symptom

User shows a screenshot of LM Studio's sidebar with a "Show Chat" option
(visible when LM Studio's built-in chat panel is collapsed). They believe
this is where they should be chatting with Hermes.

## Diagnosis

The user is confusing LM Studio's built-in chat UI with Hermes' chat
interface. LM Studio has TWO modes:

1. **Chat tab** — LM Studio's own built-in chat. NOT used with Hermes.
2. **Local Server tab** — Starts an OpenAI-compatible API server on port 1234.
   This is what Hermes connects to.

The "Show Chat" button controls visibility of LM Studio's built-in chat
window (mode 1). It has nothing to do with Hermes.

## The Fix (Explained to User)

```
┌─────────────────────────────────────────────┐
│  LM Studio (the app)                        │
│                                             │
│  ┌─────────────┐   ┌──────────────────────┐ │
│  │ Chat tab     │   │ Local Server tab     │ │  ← Use THIS one
│  │ (ignore it)  │   │ (click "Start        │ │
│  │              │   │  Server" here)       │ │
│  └─────────────┘   └──────────┬───────────┘ │
└───────────────────────────────┼─────────────┘
                                │ API (port 1234)
                                ▼
┌─────────────────────────────────────────────┐
│  Hermes Agent                               │
│  You chat via CLI, Discord, Telegram, TUI   │
└─────────────────────────────────────────────┘
```

1. Close/hide LM Studio's built-in Chat panel (the "Show Chat" button is irrelevant)
2. In LM Studio, go to the **Local Server** tab
3. Click **"Start Server"** — this starts the API on port 1234
4. In Hermes: `hermes config set provider lmstudio`
5. Chat with Hermes via CLI (`hermes`), Discord, or Telegram — never through LM Studio's chat

## Verification

```bash
curl http://127.0.0.1:1234/v1/models
```

Must return JSON with loaded models. If not, the server isn't running.

## Key Insight for Support

The user confused "Show Chat" (restoring LM Studio's own UI panel) with
"connecting to the API server." The architecture explanation with the ASCII
diagram resolves this quickly. Always pair with the verification curl command.
