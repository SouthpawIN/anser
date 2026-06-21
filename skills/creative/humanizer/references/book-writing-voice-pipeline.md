# Book Writing with Hermes: Voice Pipeline

## Overview

Teaching Hermes your writing voice for long-form book projects is not a single-skill operation. It's a **three-stage pipeline**:

```
Stage 1: Drafting          Stage 2: Humanizer    Stage 3: Final Proof
(Custom skill)          →   (Strip AI-isms)   →   (Voice validation)
```

Each stage has a distinct responsibility. Mixing them up (e.g. expecting the humanizer to *add* voice) produces bland output.

## The Four Voice Mechanisms

| Mechanism | Purpose | Persistence |
|---|---|---|
| **Custom book-writing skill** | Defines workflow + embeds voice samples + style rules | Until deleted |
| **Humanizer skill** | Strips AI-isms after drafting (29 patterns) | Bundled with Hermes |
| **Memory / user profile** | Auto-accumulates style corrections across sessions | Permanent |
| **Final proof skill** | QA validates output against style rules | Custom, optional |

## Stage 1: Custom Book-Writing Skill (Primary)

The drafting stage must carry the voice. A custom skill contains:

- **Workflow definition** — outline → chunked drafting → assemble → revise
- **Style rules** — explicit sentence length, banned phrases, preferred constructions
- **Voice samples** — embedded or referenced prose excerpts
- **Project context** — characters, setting, tone guide

### Example structure

```yaml
# ~/.hermes/skills/my-book-project/SKILL.md
---
name: my-book-project
description: "Draft my sci-fi novel with my voice"
tags: [writing, creative, long-form]
---

## Voice Rules
- Sentences: 15-25 words, mixed length
- Dialogue tags: "said" / "asked" only — no adverbs
- No passive voice in action scenes
- No semicolons; use em dashes or periods

## Banned Phrases
- "in order to" → "to"
- "it is worth noting" — delete entirely
- "delve into", "tapestry", "showcase"

## Samples
See ~/writing/my_voice_sample.txt

## Workflow
1. Outline chapter (user provides premise)
2. Draft in 800-word sections
3. After each section: "does this match the voice sample?"
4. Assemble full chapter
5. Humanizer pass
6. Proof pass
```

## Stage 2: Humanizer (Post-Processing Filter)

This strips 29 AI writing patterns (em dash overuse, rule-of-three, -ing endings, "serves as / stands as", elegant variation, etc.). It is a **negative filter** — it removes bad patterns but cannot inject your specific voice.

**How to run it:**
```
/skill humanizer
"Humanize chapter-12.md using my voice from ~/writing/my_voice_sample.txt as reference"
```

The humanizer reads the sample, analyzes sentence length, word choice, punctuation habits, and paragraph structure, then matches the rewrite.

## Stage 3: Final Proof Skill (Validation)

A custom skill that acts as a QA gate. It doesn't generate — it validates:

- Banned phrase scan
- Sentence length distribution
- Passive voice ratio
- AI-ism pattern count
- Consistency with previous chapters

## Common Mistakes

| Mistake | Why it fails |
|---|---|
| Relying only on humanizer | Can't add voice — only remove bad patterns |
| No voice sample file | Hermes needs concrete reference prose |
| One-shotting whole chapters | Voice drifts over 5000+ word generations |
| No final proof pass | Chapters 1 and 20 will read like different authors |

## Example Pipeline Command

```
/skill my-book-project
/skill humanizer
"Draft chapter 12. Scene: [premise]. Characters: [X, Y].
 Match the voice in ~/writing/my_voice_sample.txt.
 After drafting, humanize the output.
 Then validate against our style guide."
```

## GPT 5.5 via Codex OAuth

GPT 5.5 is available through the OpenAI Codex OAuth provider (requires ChatGPT Pro/Plus). Set it up:

```bash
hermes model
# → "OpenAI Codex" → "gpt-5.5"
```

Context window: 272K via Codex OAuth. Style adherence is stronger than GPT-4. The skill architecture works with any provider — style rules transfer between models.