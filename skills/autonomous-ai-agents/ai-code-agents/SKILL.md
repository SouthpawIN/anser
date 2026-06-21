---
name: ai-code-agents
description: "Consolidated umbrella for autonomous AI coding agents and multi-agent workflows: Codex, OpenCode, Claude Code, Hermes Agent, Kanban orchestration, multi-user deployment."
author: Hermes Agent
version: 2.0.0
license: MIT
tags: ["Umbrella", "Autonomous Ai Agents", "Consolidated", "Claude Code", "Hermes", "Kanban", "Multi-Agent"]
related_skills: ["codex", "opencode", "claude-code", "hermes-agent", "hermes-multi-user-deployment", "kanban-orchestrator", "kanban-worker", "anser"]
---

# Ai Code Agents

## Codex CLI


# Codex CLI

Delegate coding tasks to [Codex](https://github.com/openai/codex) via the Hermes terminal. Codex is OpenAI's autonomous coding agent CLI.

## When to use

- Building features
- Refactoring
- PR reviews
- Batch issue fixing

Requires the codex CLI and a git repository.

## Prerequisites

- Codex installed: `npm install -g @openai/codex`
- OpenAI API key configured
- **Must run inside a git repository** — Codex refuses to run outside one
- Use `pty=true` in terminal calls — Codex is an interactive terminal app

## One-Shot Tasks

```
terminal(command="codex exec 'Add dark mode toggle to settings'", workdir="~/project", pty=true)
```

For scratch work (Codex needs a git repo):
```
terminal(command="cd $(mktemp -d) && git init && codex exec 'Build a snake game in Python'", pty=true)
```

## Background Mode (Long Tasks)

```
# Start in background with PTY
terminal(command="codex exec --full-auto 'Refactor the auth module'", workdir="~/project", background=true, pty=true)
# Returns session_id

# Monitor progress
process(action="poll", session_id="<id>")
process(action="log", session_id="<id>")

# Send input if Codex asks a question
process(action="submit", session_id="<id>", data="yes")

# Kill if needed
process(action="kill", session_id="<id>")
```

## Key Flags

| Flag | Effect |
|------|--------|
| `exec "prompt"` | One-shot execution, exits when done |
| `--full-auto` | Sandboxed but auto-approves file changes in workspace |
| `--yolo` | No sandbox, no approvals (fastest, most dangerous) |

## PR Reviews

Clone to a temp directory for safe review:

```
terminal(command="REVIEW=$(mktemp -d) && git clone https://github.com/user/repo.git $REVIEW && cd $REVIEW && gh pr checkout 42 && codex review --base origin/main", pty=true)
```

## Parallel Issue Fixing with Worktrees

```
# Create worktrees
terminal(command="git worktree add -b fix/issue-78 /tmp/issue-78 main", workdir="~/project")
terminal(command="git worktree add -b fix/issue-99 /tmp/issue-99 main", workdir="~/project")

# Launch Codex in each
terminal(command="codex --yolo exec 'Fix issue #78: <description>. Commit when done.'", workdir="/tmp/issue-78", background=true, pty=true)
terminal(command="codex --yolo exec 'Fix issue #99: <description>. Commit when done.'", workdir="/tmp/issue-99", background=true, pty=true)

# Monitor
process(action="list")

# After completion, push and create PRs
terminal(command="cd /tmp/issue-78 && git push -u origin fix/issue-78")
terminal(command="gh pr create --repo user/repo --head fix/issue-78 --title 'fix: ...' --body '...'")

# Cleanup
terminal(command="git worktree remove /tmp/issue-78", workdir="~/project")
```

## Batch PR Reviews

```
# Fetch all PR refs
terminal(command="git fetch origin '+refs/pull/*/head:refs/remotes/origin/pr/*'", workdir="~/project")

# Review multiple PRs in parallel
terminal(command="codex exec 'Review PR #86. git diff origin/main...origin/pr/86'", workdir="~/project", background=true, pty=true)
terminal(command="codex exec 'Review PR #87. git diff origin/main...origin/pr/87'", workdir="~/project", background=true, pty=true)

# Post results
terminal(command="gh pr comment 86 --body '<review>'", workdir="~/project")
```

## Rules

1. **Always use `pty=true`** — Codex is an interactive terminal app and hangs without a PTY
2. **Git repo required** — Codex won't run outside a git directory. Use `mktemp -d && git init` for scratch
3. **Use `exec` for one-shots** — `codex exec "prompt"` runs and exits cleanly
4. **`--full-auto` for building** — auto-approves changes within the sandbox
5. **Background for long tasks** — use `background=true` and monitor with `process` tool
6. **Don't interfere** — monitor with `poll`/`log`, be patient with long-running tasks
7. **Parallel is fine** — run multiple Codex processes at once for batch work


## OpenCode CLI


# OpenCode CLI

Use [OpenCode](https://opencode.ai) as an autonomous coding worker orchestrated by Hermes terminal/process tools. OpenCode is a provider-agnostic, open-source AI coding agent with a TUI and CLI.

## When to Use

- User explicitly asks to use OpenCode
- You want an external coding agent to implement/refactor/review code
- You need long-running coding sessions with progress checks
- You want parallel task execution in isolated workdirs/worktrees

## Prerequisites

- OpenCode installed: `npm i -g opencode-ai@latest` or `brew install anomalyco/tap/opencode`
- Auth configured: `opencode auth login` or set provider env vars (OPENROUTER_API_KEY, etc.)
- Verify: `opencode auth list` should show at least one provider
- Git repository for code tasks (recommended)
- `pty=true` for interactive TUI sessions

## Binary Resolution (Important)

Shell environments may resolve different OpenCode binaries. If behavior differs between your terminal and Hermes, check:

```
terminal(command="which -a opencode")
terminal(command="opencode --version")
```

If needed, pin an explicit binary path:

```
terminal(command="$HOME/.opencode/bin/opencode run '...'", workdir="~/project", pty=true)
```

## One-Shot Tasks

Use `opencode run` for bounded, non-interactive tasks:

```
terminal(command="opencode run 'Add retry logic to API calls and update tests'", workdir="~/project")
```

Attach context files with `-f`:

```
terminal(command="opencode run 'Review this config for security issues' -f config.yaml -f .env.example", workdir="~/project")
```

Show model thinking with `--thinking`:

```
terminal(command="opencode run 'Debug why tests fail in CI' --thinking", workdir="~/project")
```

Force a specific model:

```
terminal(command="opencode run 'Refactor auth module' --model openrouter/anthropic/claude-sonnet-4", workdir="~/project")
```

## Interactive Sessions (Background)

For iterative work requiring multiple exchanges, start the TUI in background:

```
terminal(command="opencode", workdir="~/project", background=true, pty=true)
# Returns session_id

# Send a prompt
process(action="submit", session_id="<id>", data="Implement OAuth refresh flow and add tests")

# Monitor progress
process(action="poll", session_id="<id>")
process(action="log", session_id="<id>")

# Send follow-up input
process(action="submit", session_id="<id>", data="Now add error handling for token expiry")

# Exit cleanly — Ctrl+C
process(action="write", session_id="<id>", data="\x03")
# Or just kill the process
process(action="kill", session_id="<id>")
```

**Important:** Do NOT use `/exit` — it is not a valid OpenCode command and will open an agent selector dialog instead. Use Ctrl+C (`\x03`) or `process(action="kill")` to exit.

### TUI Keybindings

| Key | Action |
|-----|--------|
| `Enter` | Submit message (press twice if needed) |
| `Tab` | Switch between agents (build/plan) |
| `Ctrl+P` | Open command palette |
| `Ctrl+X L` | Switch session |
| `Ctrl+X M` | Switch model |
| `Ctrl+X N` | New session |
| `Ctrl+X E` | Open editor |
| `Ctrl+C` | Exit OpenCode |

### Resuming Sessions

After exiting, OpenCode prints a session ID. Resume with:

```
terminal(command="opencode -c", workdir="~/project", background=true, pty=true)  # Continue last session
terminal(command="opencode -s ses_abc123", workdir="~/project", background=true, pty=true)  # Specific session
```

## Common Flags

| Flag | Use |
|------|-----|
| `run 'prompt'` | One-shot execution and exit |
| `--continue` / `-c` | Continue the last OpenCode session |
| `--session <id>` / `-s` | Continue a specific session |
| `--agent <name>` | Choose OpenCode agent (build or plan) |
| `--model provider/model` | Force specific model |
| `--format json` | Machine-readable output/events |
| `--file <path>` / `-f` | Attach file(s) to the message |
| `--thinking` | Show model thinking blocks |
| `--variant <level>` | Reasoning effort (high, max, minimal) |
| `--title <name>` | Name the session |
| `--attach <url>` | Connect to a running opencode server |

## Procedure

1. Verify tool readiness:
   - `terminal(command="opencode --version")`
   - `terminal(command="opencode auth list")`
2. For bounded tasks, use `opencode run '...'` (no pty needed).
3. For iterative tasks, start `opencode` with `background=true, pty=true`.
4. Monitor long tasks with `process(action="poll"|"log")`.
5. If OpenCode asks for input, respond via `process(action="submit", ...)`.
6. Exit with `process(action="write", data="\x03")` or `process(action="kill")`.
7. Summarize file changes, test results, and next steps back to user.

## PR Review Workflow

OpenCode has a built-in PR command:

```
terminal(command="opencode pr 42", workdir="~/project", pty=true)
```

Or review in a temporary clone for isolation:

```
terminal(command="REVIEW=$(mktemp -d) && git clone https://github.com/user/repo.git $REVIEW && cd $REVIEW && opencode run 'Review this PR vs main. Report bugs, security risks, test gaps, and style issues.' -f $(git diff origin/main --name-only | head -20 | tr '\n' ' ')", pty=true)
```

## Parallel Work Pattern

Use separate workdirs/worktrees to avoid collisions:

```
terminal(command="opencode run 'Fix issue #101 and commit'", workdir="/tmp/issue-101", background=true, pty=true)
terminal(command="opencode run 'Add parser regression tests and commit'", workdir="/tmp/issue-102", background=true, pty=true)
process(action="list")
```

## Session & Cost Management

List past sessions:

```
terminal(command="opencode session list")
```

Check token usage and costs:

```
terminal(command="opencode stats")
terminal(command="opencode stats --days 7 --models anthropic/claude-sonnet-4")
```

## Pitfalls

- Interactive `opencode` (TUI) sessions require `pty=true`. The `opencode run` command does NOT need pty.
- `/exit` is NOT a valid command — it opens an agent selector. Use Ctrl+C to exit the TUI.
- PATH mismatch can select the wrong OpenCode binary/model config.
- If OpenCode appears stuck, inspect logs before killing:
  - `process(action="log", session_id="<id>")`
- Avoid sharing one working directory across parallel OpenCode sessions.
- Enter may need to be pressed twice to submit in the TUI (once to finalize text, once to send).

## Verification

Smoke test:

```
terminal(command="opencode run 'Respond with exactly: OPENCODE_SMOKE_OK'")
```

Success criteria:
- Output includes `OPENCODE_SMOKE_OK`
- Command exits without provider/model errors
- For code tasks: expected files changed and tests pass

## Rules

1. Prefer `opencode run` for one-shot automation — it's simpler and doesn't need pty.
2. Use interactive background mode only when iteration is needed.
3. Always scope OpenCode sessions to a single repo/workdir.
4. For long tasks, provide progress updates from `process` logs.
5. Report concrete outcomes (files changed, tests, remaining risks).
6. Exit interactive sessions with Ctrl+C or kill, never `/exit`.



## Claude Code

> **Source:** Absorbed from `claude-code` skill. Delegate coding tasks to [Claude Code](https://code.claude.com/docs/en/cli-reference) (Anthropic's autonomous coding agent CLI) via the Hermes terminal.

### When to Use
- Building features, refactoring, PR reviews
- Long-running coding tasks with progress monitoring
- Multi-turn interactive sessions
- Worktree-based parallel development

### Prerequisites
- Install: `npm install -g @anthropic-ai/claude-code`
- Auth: `claude` once to log in, or `ANTHROPIC_API_KEY`
- Console auth: `claude auth login --console` for API key billing
- Check: `claude auth status`, `claude doctor`, `claude --version` (v2.x+ required)

### Two Orchestration Modes

**Mode 1: Print Mode (`-p`) — Non-Interactive (PREFERRED)**
```bash
terminal(command="claude -p 'Add error handling to all API calls in src/' --allowedTools 'Read,Edit' --max-turns 10", workdir="/path/to/project", timeout=120)
```
- One-shot, returns result and exits
- No PTY needed, skips all interactive dialogs
- Use `--output-format json` for structured output
- Use `--max-turns` and `--max-budget-usd` to prevent runaway

**Mode 2: Interactive PTY via tmux — Multi-Turn**
```bash
terminal(command="tmux new-session -d -s claude-work -x 140 -y 40")
terminal(command="tmux send-keys -t claude-work 'cd /path/to/project && claude' Enter")
terminal(command="sleep 5 && tmux send-keys -t claude-work 'Refactor auth module' Enter")
terminal(command="sleep 15 && tmux capture-pane -t claude-work -p -S -50")
terminal(command="tmux send-keys -t claude-work '/exit' Enter")
```
- Full conversational REPL, slash commands (`/compact`, `/review`, `/model`)
- Handle workspace trust dialog (Enter) and permissions dialog (Down+Enter)

### Key Features
- **Worktree mode**: `claude -w feature-x --tmux` creates isolated git worktree + tmux
- **PR review**: `claude -p 'Review PR' --from-pr 42` or `claude -w pr-review --tmux`
- **Parallel instances**: Multiple tmux sessions for parallel tasks
- **Session resumption**: `claude -c`, `claude -r <id>`, `--fork-session`

### Rules for Hermes
1. Prefer print mode (`-p`) for single tasks
2. Use tmux for multi-turn interactive work
3. Always set `workdir` and `--max-turns` in print mode
4. Monitor tmux sessions with `tmux capture-pane`
5. Clean up tmux sessions when done


## Hermes Agent

> **Source:** Absorbed from `hermes-agent` skill. Configure, extend, or contribute to Hermes Agent.

### When to Use
- Setting up, configuring, or troubleshooting Hermes Agent
- Spawning additional Hermes instances for multi-agent workflows
- Managing skills, tools, MCP servers, profiles, cron jobs
- Operating the messaging gateway (Telegram, Discord, Slack, etc.)

### Quick Start
```bash
# Install
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# Interactive chat
hermes

# Single query
hermes chat -q "What is the capital of France?"

# Setup wizard
hermes setup
```

### Key CLI Commands
| Command | Purpose |
|---------|---------|
| `hermes chat -q "..."` | Single query, non-interactive |
| `hermes model` | Interactive model/provider picker |
| `hermes config set KEY VAL` | Set config value |
| `hermes tools` | Enable/disable toolsets |
| `hermes skills install ID` | Install skill from hub or URL |
| `hermes mcp add NAME` | Add MCP server |
| `hermes gateway run` | Start messaging gateway |
| `hermes kanban create TITLE` | Create Kanban task |
| `hermes profile create NAME` | Create new profile |
| `hermes cron create SCHEDULE` | Create scheduled job |

### Slash Commands (In-Session)
| Command | Purpose |
|---------|---------|
| `/new` / `/reset` | Fresh session |
| `/model [name]` | Show/change model |
| `/skills` | Search/install skills |
| `/cron` | Manage cron jobs |
| `/kanban` | Kanban board commands |
| `/background <prompt>` | Fire-and-forget background task |
| `/personality [name]` | Set personality |
| `/yolo` | Toggle approval bypass |

### Multi-Agent Orchestration (Kanban)
```bash
# Create specialist profiles
hermes profile create researcher
hermes profile create writer
hermes profile create backend-eng
hermes profile create reviewer

# Set distinct SOUL.md prompts per profile
echo "You are a research specialist..." > ~/.hermes/profiles/researcher/SOUL.md

# Init kanban and start gateway (runs dispatcher)
hermes kanban init
hermes gateway start

# Orchestrator creates task graph with dependencies
```

### Spawning Additional Hermes Instances
```bash
# One-shot
terminal(command="hermes chat -q 'Research GRPO papers'", timeout=300)

# Interactive PTY via tmux
terminal(command="tmux new-session -d -s agent1 -x 120 -y 40 'hermes'")
terminal(command="sleep 8 && tmux send-keys -t agent1 'Build FastAPI auth service' Enter")
terminal(command="sleep 20 && tmux capture-pane -t agent1 -p")
```

### Key Paths
- Config: `~/.hermes/config.yaml`
- Secrets: `~/.hermes/.env`
- Skills: `$HERMES_HOME/skills/`
- Sessions: `~/.hermes/sessions/`

### Subagent Delegation & Model Routing
See `references/subagent-delegation-patterns.md` for patterns covering:
- Skills don't inherit to subagents (embed in context vs give `skills` toolset)
- Model routing per subagent via `delegation.model` / `delegation.provider`
- Context as the only parent→child information channel
- Approval modes for headless subagents (`approvals.mode: smart`, `command_allowlist`)


## Hermes Multi-User Deployment

> **Source:** Absorbed from `hermes-multi-user-deployment` skill. Multi-user/multi-profile Hermes Agent deployment patterns.

### When to Use
- Running Hermes for multiple users on one machine
- Isolating configs, sessions, skills per user/project
- Shared gateway with per-user authentication

### Deployment Patterns

**Pattern 1: Profile Per User**
```bash
hermes profile create alice
hermes profile create bob
# Each gets ~/.hermes/profiles/<name>/ with isolated config, skills, memory
```

**Pattern 2: Shared Gateway, Per-User Profiles**
```bash
# Gateway runs as service, routes to user's profile based on platform user ID
hermes gateway install
hermes gateway start
```

**Pattern 3: Docker Compose Fleet**
```yaml
# docker-compose.yml
services:
  hermes-alice:
    image: nousresearch/hermes-agent
    environment:
      - HERMES_PROFILE=alice
    volumes:
      - ./data/alice:/root/.hermes
  hermes-bob:
    image: nousresearch/hermes-agent
    environment:
      - HERMES_PROFILE=bob
    volumes:
      - ./data/bob:/root/.hermes
```

### Key Considerations
- Profiles isolate: config, skills, sessions, memory, cron, plugins
- Gateway can serve multiple profiles simultaneously
- Use `HERMES_PROFILE` env var or `--profile` flag to select


## Kanban Orchestrator

> **Source:** Absorbed from `kanban-orchestrator` skill. Decomposition playbook + anti-temptation rules for orchestrator profile routing work through Kanban.

### When to Use Kanban Board
Create Kanban tasks when:
1. Multiple specialists needed (research + analysis + writing)
2. Work should survive crash/restart
3. Human-in-the-loop expected at any step
4. Multiple subtasks can run in parallel
5. Review/iteration expected
6. Audit trail matters

### Anti-Temptation Rules
- **Do not execute the work yourself** — route, don't execute
- **For any concrete task, create a Kanban task and assign it**
- **Split multi-lane requests before creating cards**
- **Run independent lanes in parallel**
- **Never create dependent work as independent ready cards** — use `parents=[...]`
- **If no specialist fits, ask user which profile to use**

### Decomposition Playbook
1. **Understand the goal** — ask clarifying questions
2. **Sketch the task graph** — extract lanes, map to profiles, decide dependencies
3. **Create tasks and link** — use `kanban_create` with `parents=[...]` for dependencies
4. **Complete your own task** with summary and `metadata.task_graph`
5. **Report back to user** in plain prose

### Example Task Graph
```python
t1 = kanban_create(title="research: cost comparison", assignee="researcher", body="Compare infrastructure costs...")
t2 = kanban_create(title="research: performance comparison", assignee="researcher", body="Compare query latency...")
t3 = kanban_create(title="synthesize recommendation", assignee="analyst", body="Read T1 and T2...", parents=[t1, t2])
t4 = kanban_create(title="draft decision memo", assignee="writer", body="Turn T3 into CTO memo...", parents=[t3])
```

### Common Patterns
- **Fan-out + fan-in**: N research cards → 1 synthesis card
- **Parallel implementation + validation**: 1 implementer + 1 explorer → 1 reviewer
- **Pipeline with gates**: planner → implementer → reviewer (each `parents=[previous]`)
- **Same-profile queue**: N tasks to same profile, dispatcher serializes
- **Human-in-the-loop**: Any task can `kanban_block()` to wait for input


## Kanban Worker

> **Source:** Absorbed from `kanban-worker` skill. Pitfalls, examples, and edge cases for Hermes Kanban workers.

### Workspace Handling
| Kind | Description |
|------|-------------|
| `scratch` | Fresh tmp dir, yours alone |
| `dir:<path>` | Shared persistent directory |
| `worktree` | Git worktree at resolved path |

### Tenant Isolation
If `$HERMES_TENANT` set, prefix memory entries: `business-a: Acme is our biggest customer`

### Good Summary + Metadata
```python
# Coding task
kanban_complete(summary="shipped rate limiter — token bucket, 14 tests pass", metadata={"changed_files": [...], "tests_run": 14, "tests_passed": 14})

# Review task (block for human review)
kanban_comment(body="review-required handoff: " + json.dumps({"changed_files": [...], "diff_path": "..."}))
kanban_block(reason="review-required: rate limiter shipped, needs eyes on user_id/IP fallback choice")
```

### Claiming Created Cards
```python
c1 = kanban_create(title="remediate SQL injection", assignee="security-worker")
c2 = kanban_create(title="fix CSRF middleware", assignee="web-worker")
kanban_complete(summary="Review done; spawned remediations.", created_cards=[c1["task_id"], c2["task_id"]])
```

### Block Reasons That Get Answered Fast
```python
kanban_comment(body="Full context: I have user IPs from Cloudflare headers but some users are behind NATs...")
kanban_block(reason="Rate limit key choice: IP (simple, NAT-unsafe) or user_id (requires auth, skips anonymous)?")
```

### Tool Availability Matrix — Workers vs Orchestrators

Workers (dispatcher-spawned, `HERMES_KANBAN_TASK` set) have a restricted toolset:

| Tool | Worker | Orchestrator | Notes |
|------|--------|-------------|-------|
| `kanban_show` | ✅ | ✅ | Read any task |
| `kanban_comment` | ✅ | ✅ | Comment on ANY task — the inter-agent comm channel |
| `kanban_create` | ✅ | ✅ | Spawn child tasks with any assignee |
| `kanban_block` | ✅ (own only) | ✅ | `_enforce_worker_task_ownership` blocks foreign task IDs |
| `kanban_complete` | ✅ (own only) | ✅ | Same ownership guard |
| `kanban_heartbeat` | ✅ (own only) | ✅ | Same |
| `kanban_link` | ✅ | ✅ | Add parent→child edges post-creation |
| `kanban_list` | ❌ | ✅ | Orchestrator-only (`_check_kanban_orchestrator_mode`) |
| `kanban_unblock` | ❌ | ✅ | **Orchestrator-only** — workers CANNOT unblock tasks |

This is a security boundary, not a limitation: prevents prompt-injected workers from unblocking themselves or peers.

### Sticky Blocks

When a worker calls `kanban_block`, the block is **sticky** — `recompute_ready()` will NOT auto-promote the task even if parents complete. Only an explicit `kanban_unblock` (orchestrator/human) can flip it back. The mechanism: `_has_sticky_block()` checks if the most recent `blocked`/`unblocked` event is `blocked`.

### Sending Work Back Upstream (Reviewer → Researcher/Writer)

A reviewer worker that finds issues in upstream work cannot unblock itself, and cannot directly "send" work back. The correct pattern:

1. **Reviewer creates rework child tasks** via `kanban_create` assigned to upstream profiles
2. **Reviewer comments on its own task** with full context and rework task IDs
3. **Reviewer blocks its own task** with reason referencing the rework tasks
4. **Rework tasks dispatch and complete** (researcher/writer do the work)
5. **Orchestrator (or human) calls `kanban_unblock`** on the reviewer's task when rework is done
6. **Reviewer re-spawns** — sees prior block reason + rework summaries in `build_worker_context`

### Alternative: Complete-and-Recreate (No Orchestrator Needed)

If no orchestrator is available, the reviewer can:
1. `kanban_complete` its current task with a "rejected" summary
2. `kanban_create` rework tasks for upstream profiles
3. `kanban_create` a NEW re-review task with `parents=[rework_tasks]`
4. `kanban_link(parent_id=new_review, child_id=downstream_task)` to gate downstream work on the re-review

This avoids blocking entirely but creates more tasks on the board.

### Do NOT
- Call `delegate_task` instead of `kanban_create` (delegate_task = ephemeral RPC)
- Call `clarify` (headless, no live user) — use `kanban_comment` + `kanban_block`
- Create follow-up tasks assigned to yourself — assign to right specialist
- Complete a task you didn't actually finish — block it instead
- Create dependent work as independent ready cards — use `parents=[...]`
- Expect a worker to call `kanban_unblock` — it's orchestrator-only by design
### Pitfalls: Dangerous Command Approval Hangs
Kanban workers/subagents run headlessly — there's no user to approve dangerous-command prompts, so commands containing IP addresses, `rm -rf`, or other flagged patterns hang until timeout (60s default) then get denied. Fix with one or both of these in `~/.hermes/config.yaml`:

```yaml
# Pre-approve known-safe patterns (IPs, common commands)
command_allowlist:
  - "192.168.1."
  - "10.0.0."
  - "curl.*localhost"

# Or switch to AI-powered approval — auto-approves low-risk, blocks dangerous
approvals:
  mode: smart
```

When a worker's task involves network calls, SSH, or file cleanup, add those patterns to `command_allowlist` or use `approvals.mode: smart` to prevent headless hangs.


## Anser

> **Source:** Absorbed from `anser` skill. Hermes Agent tech-support workflow.

### When to Use
- Troubleshooting Hermes Agent issues
- Diagnosing configuration, model, or tool problems
- User reports "Hermes not working" or unexpected behavior

### Workflow
1. **Run `hermes doctor [--fix]`** — checks dependencies, config, credentials
2. **Check `hermes status [--all]`** — component health (tools, skills, MCP, gateway)
3. **Verify auth** — `hermes auth list`, `hermes login` for OAuth providers
4. **Check logs** — `~/.hermes/logs/` (gateway.log, error.log)
5. **Common fixes**:
   - Tool not available → `hermes tools` enable it
   - Model issues → `hermes model` or check `.env` API keys
   - Gateway issues → `hermes gateway restart` or check `gateway.log`
   - Skills not showing → `hermes skills config` check platform enablement

### Quick Diagnostic Commands
```bash
hermes doctor --fix
hermes status --all
hermes config check
hermes tools list
hermes skills list
hermes profile list        # Shows all profiles + gateway status
```

### Profile Management (v0.16.0+)

**Architecture reference**: See `references/hermes-profile-architecture.md` for full internal architecture details from source code.
**Discord bot template**: See `templates/discord-bot-profile.yaml` for a ready-to-copy profile config.
**Multi-bot tutorial**: See `references/multi-discord-bot-setup.md` for complete step-by-step.

#### Dashboard Unified Management

After `hermes update`, the top-left of the web dashboard (port 9119) shows a profile selector dropdown. One dashboard instance manages ALL profiles — **do not run a second dashboard instance** (it will conflict on port 9119).

- Switch profiles from the dropdown to instantly see that profile's config, skills, sessions, env, model, cron, logs
- No restart, no separate browser tab, no separate port
- The dashboard backend reads `~/.hermes/profiles/<name>/config.yaml` for the selected profile
- Profiles page has full card-based UI: create, rename, delete, set active, edit model/description/SOUL

#### CLI Commands

```bash
hermes profile list              # All profiles + model + gateway status (◆ = active)
hermes profile create <name>     # New profile (--clone-all, --no-skills, --description, --model)
hermes profile use <name>        # Set as active (permanent switch)
hermes profile show/describe/rename/delete <name>
hermes profile export/import     # tar.gz portability
hermes profile install <url>     # From git URL (distributions)
hermes profile update/info <name>   # Re-pull or inspect distribution metadata
```

#### Multi-Profile Gateway Management

Each profile runs its own gateway process with its own PID. Manage them per-profile:

```bash
# Install gateway service for a profile
hermes profile use discord-mod && hermes gateway install

# Start/stop per profile
hermes profile use discord-mod && hermes gateway start
hermes profile use discord-mod && hermes gateway stop

# List all profiles + gateway status
hermes gateway list

# Stop ALL gateways
hermes gateway stop --all
```

For systemd: `hermes-gateway-<profile>.service` units. For foreground: `hermes profile use <name> && hermes gateway run`.

#### Multi-Discord-Bot Deployment Pattern

Each profile gets its own `DISCORD_BOT_TOKEN` in `.env`. Steps:

1. Create two Discord applications at https://discord.com/developers/applications (separate tokens)
2. `hermes profile create discord-mod --model ... --description "Moderation bot"`
3. `hermes profile create discord-helper --model ... --description "Community helper"`
4. Set `DISCORD_BOT_TOKEN=***` in each profile's `.env`
5. Configure Discord settings per profile in `config.yaml` (require_mention, personality, toolsets)
6. Install and start gateway per profile
7. Manage both from ONE dashboard — switch profiles in the dropdown

**Full tutorial**: See `references/multi-discord-bot-setup.md`

#### Pitfalls

- **"I need a second dashboard"** — No. That was the old way. The new unified dashboard (v0.16.0+) manages all profiles from one instance. A second `hermes dashboard` will conflict on port 9119.
- **Profile not showing in dashboard dropdown** — Must run `hermes update`, clear browser cache, and have >1 profile (`hermes profile create` if needed)
- **Can't delete profile** — Stop its gateway first: `hermes profile use <name> && hermes gateway stop` then `hermes profile delete <name>`
- **Model credentials** — If using `hermes setup --portal` (OAuth), all profiles inherit provider access from default. No need to set API keys per profile.
- **Dashboard shows wrong settings** — Check which profile is selected in the top-left dropdown
- **Desktop sessions not showing for a profile** — In global remote mode, all profiles route through one backend with `?profile=` param; check Settings → Gateway

### Escalation Path
1. User runs diagnostics above
2. If unresolved, collect: `hermes doctor` output, config.yaml (sanitized), relevant logs
3. Report to Nous Research with reproduction steps


