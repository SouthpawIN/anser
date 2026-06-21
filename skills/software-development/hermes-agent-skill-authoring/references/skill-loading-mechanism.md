# How Hermes Loads and Discovers Skills at Runtime

Sourced from source-code analysis (gateway/run.py, discord adapter,
AGENTS.md, CONTRIBUTING.md). This is the mechanism that answers: "how does
Hermes decide which skills to load when the user sends a message?"

---

## Three-Layer Skill Resolution

### Layer 1: System Prompt `<available_skills>` Scanning (Primary)

Every turn sent to the model includes an `<available_skills>` block listing
all installed skills with names and descriptions. The prompt instructs:

> "Before replying, scan the skills below. If a skill matches or is even
> partially relevant to your task, you MUST load it with skill_view(name)
> and follow its instructions."

**The LLM itself is the parser.** No separate keyword-matching engine, no
regex triggers, no NLP pre-processor. The LLM reads the user's message,
semantically matches it against skill descriptions, and loads what's
relevant. This works across ALL platforms (CLI, Discord, Telegram, Slack).

Key implication for skill authors: descriptions MUST be rich enough for
semantic matching. "Use when debugging X" is good; "Debug X" is not.

### Layer 2: Channel/Topic Auto-Skill Bindings (Gateway Only)

Platform adapters inject skills based on pre-configured channel/topic
bindings. This happens BEFORE the LLM sees the message.

**Discord** — `channel_skill_bindings` in platform config extra:
```yaml
channel_skill_bindings:
  - id: "1234567890"
    skills: ["github-developer-tools"]
```

**Telegram** — `dm_topics` skill binding:
```yaml
dm_topics:
  - topics:
      - name: "Code Review"
        skill: "code-quality"
```

Code path: adapter populates `MessageEvent.auto_skill` → `gateway/run.py`
(line 8138) injects skill content as synthetic user message on **new
sessions only** (re-injection breaks prompt caching).

### Layer 3: Explicit `/skill` Slash Commands

Users manually load skills via `/skillname` in messaging platforms.
`agent/skill_commands.py` scans `~/.hermes/skills/` and injects the skill
content as a synthetic user message.

---

## Prompt Caching Constraint

From `AGENTS.md` (line 370):

> "Skill slash commands: `agent/skill_commands.py` scans `~/.hermes/skills/`,
> injects as **user message** (not system prompt) to preserve prompt caching"

The system prompt must remain **byte-stable** for the life of a
conversation. Mutating it invalidates the cache and multiplies API costs.
Therefore, ALL skill content (from layers 2 and 3) is injected as user
messages — never as system prompt mutations.

Layer 1 (system prompt scanning) doesn't mutate the system prompt either —
the `<available_skills>` block is part of the frozen system prompt, and
`skill_view()` results arrive as tool results.

---

## Platform Filtering

Skills with a `platforms` frontmatter field are **hidden** from the
`<available_skills>` block, `skills_list()`, and slash commands on
incompatible platforms (CONTRIBUTING.md line 392). Apple skills won't
appear on Linux.

---

## Skill Visibility Flow

```
User sends message
  │
  ├─► Layer 2: Channel binding? → inject as user message (new sessions only)
  │
  └─► Layer 1: LLM scans <available_skills> → semantic match → skill_view()
        │
        └─► Layer 3: /skill slash command? → inject as user message

Result: Skill content enters context as tool result (layer 1) or user
message (layers 2, 3). System prompt stays frozen throughout.
```

---

## Source References

| Component | File | Line |
|-----------|------|------|
| Gateway auto-skill injection | `gateway/run.py` | 8134-8168 |
| Discord channel bindings | `plugins/platforms/discord/adapter.py` | 3965-3970 |
| Telegram DM topic bindings | `gateway/platforms/telegram.py` | 6470 |
| Slack channel bindings | `gateway/platforms/slack.py` | 2810 |
| MessageEvent.auto_skill field | `gateway/platforms/base.py` | 1451-1453 |
| Platform filtering | `CONTRIBUTING.md` | 392 |
| Slash command injection | `agent/skill_commands.py` | — |
| Prompt caching constraint | `AGENTS.md` | 370 |
