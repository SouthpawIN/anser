# Subagent Skills & Delegation Patterns

> Extracted from `tools/delegate_tool.py` source + docs, June 2026

## Core Insight: Subagents Know Nothing

Subagents start with a completely fresh conversation. They don't inherit:
- Parent's conversation history
- Parent's loaded skills
- Parent's memory
- Prior tool call results

The only context they get is the `goal` + `context` fields from `delegate_task`.

Source: `tools/delegate_tool.py` lines 933-1200 — `_build_child_agent()` calls `AIAgent()` with:
- `ephemeral_system_prompt=child_prompt` (built from goal + context only)
- `skip_context_files=True`
- `skip_memory=True`
- No skills parameter passed

## Getting Skills to Subagents

### Pattern A: Embed in `context` (Recommended for fixed workflows)

```
delegate_task(
    goal="Research X and summarize",
    context="""SKILL: [paste relevant skill instructions here]

TASK CONTEXT: [specific details about what to research]""",
    toolsets=["web"]
)
```

### Pattern B: Give `skills` toolset (Recommended for dynamic workflows)

```
delegate_task(
    goal="Research X and summarize",
    context="Load the 'web-research' skill before starting. Focus on...",
    toolsets=["web", "skills"]
)
```

The `skills` toolset gives subagents: `skills_list`, `skill_view`, `skill_manage`.
They can then call `skill_view("skill-name")` to load instructions.

## Model Routing for Subagents

Set in `~/.hermes/config.yaml`:

```yaml
delegation:
  model: "google/gemini-2.5-flash-lite"   # cheap model for all subagents
  provider: "openrouter"                    # optional — route to different provider
  max_concurrent_children: 3                # default 3
  max_iterations: 30                        # per-subagent cap
```

Subagent also gets: `reasoning_effort`, `api_key` (from parent or override), base_url inheritance.

## Blocked Toolsets (always stripped from subagents)

From `_strip_blocked_tools()` at line 727:
```python
blocked_toolset_names = {"delegation", "clarify", "memory", "code_execution"}
```

Exception: orchestrator-role subagents get `delegation` re-added at line 1028.

## Toolset Inheritance

When `toolsets` is omitted, child inherits parent's `enabled_toolsets` minus blocked ones.
When `toolsets` is provided, it's intersected with parent's available tools (subagent can't gain tools parent lacks).
Composite toolsets (e.g. `hermes-cli`) are expanded for intersection matching.
MCP toolsets are preserved by default (`inherit_mcp_toolsets: true`).

## Model Override Semantics

When `delegation.provider` is set:
- Child uses that provider instead of parent's
- Parent's `api_mode` is NOT inherited (each provider has different API surface — MiniMax uses anthropic_messages, DeepSeek uses chat_completions)
- Parent's OpenRouter provider filters (`providers_allowed`, `providers_ignored`) are cleared
- Parent's ACP transport is NOT inherited (would bypass override credentials)
- Parent's credential pool is resolved for the child's provider