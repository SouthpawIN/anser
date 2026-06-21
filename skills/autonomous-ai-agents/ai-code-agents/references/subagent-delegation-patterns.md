# Subagent Delegation Patterns — Skills, Models, Context

## Core Principle

Subagents spawned by `delegate_task` start with a **completely fresh conversation**. They have zero knowledge of the parent's conversation history, prior tool calls, loaded skills, or memory. The only context they receive is what the parent puts in the `goal` and `context` fields.

From the source (`tools/delegate_tool.py`), the child agent is constructed with:
- `skip_context_files=True` — no AGENTS.md, CLAUDE.md, project files
- `skip_memory=True` — no persistent memory
- `ephemeral_system_prompt` — only goal + context

## Pattern 1: Skills Don't Inherit — Two Solutions

### Solution A: Embed Skill Content in `context` (Fixed Workflows)

Copy relevant skill instructions directly into the `context` field:

```python
delegate_task(
    goal="Research AI agent frameworks and summarize",
    context="""SKILL: Web Research Best Practices
- Use web_search with specific queries
- Always web_extract the top 2-3 results
- Cross-reference claims across at least 2 sources
- Return findings as structured markdown with URLs

TASK: Focus on frameworks released in 2026.""",
    toolsets=["web"]
)
```

**Best for:** Known, fixed workflows where you know what skills are needed upfront.

### Solution B: Give Subagents the `skills` Toolset (Dynamic Workflows)

Let subagents load skills themselves:

```python
delegate_task(
    goal="Research AI agent frameworks",
    context="Load the 'web-research' skill before starting. Focus on 2026 releases.",
    toolsets=["web", "skills"]
)
```

The subagent calls `skill_view("web-research")` to load instructions. Tell it which skills to load in `context`.

**Best for:** Dynamic workflows where the subagent decides what it needs.

## Pattern 2: Model Routing for Token Savings

Set a cheaper/faster model for ALL subagents:

```yaml
# ~/.hermes/config.yaml
delegation:
  model: "google/gemini-2.5-flash-lite"
  provider: "openrouter"
```

For per-stage model routing (different models for research vs writing vs images), do sequential delegation — switch `delegation.model` between calls, or do high-quality tasks directly from the parent instead of delegating.

## Pattern 3: Context Is the Bridge

The `context` field is the only information channel from parent to subagent. Be thorough:

```python
# BAD
delegate_task(goal="Fix the error")

# GOOD
delegate_task(
    goal="Fix the TypeError in api/handlers.py line 47",
    context="""Project at /home/user/myproject, Python 3.11.
    File: api/handlers.py
    Error: 'NoneType' object has no attribute 'get' on line 47
    Root cause: parse_body() returns None when Content-Type is missing
    Expected fix: add guard clause before .get() call
    
    After fixing, run: pytest tests/test_handlers.py -v"""
)
```

## Pattern 4: Approval Modes for Autonomous Subagents

Subagents inherit the parent's `approvals` config. For headless/autonomous operation:

```yaml
approvals:
  mode: smart    # AI assesses risk, auto-approves low-risk commands

command_allowlist:
  - "192.168."  # Pre-approve known safe patterns
  - "curl.*api"
```

## Toolset Selection Quick Reference

| Pattern | Toolsets | Use Case |
|---------|----------|----------|
| `["web"]` | web_search, web_extract | Research, information gathering |
| `["terminal", "file"]` | shell, file ops | Code work, builds, debugging |
| `["web", "skills"]` | research + skill loading | Dynamic skill-based research |
| `["image_gen"]` | image_generate | Image generation tasks |
| `["browser"]` | browser tools | Web interaction, scraping |
