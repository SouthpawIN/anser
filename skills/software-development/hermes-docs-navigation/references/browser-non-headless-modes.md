# Browser Non-Headless Modes — Condensed Reference

> Compiled June 12, 2026 · Hermes Agent v0.16.0

## Two Paths to Visible Browser

### Path 1: CDP Attach (`/browser connect`) — Official

Start a Chromium-family browser with `--remote-debugging-port=9222`, then run `/browser connect` in the Hermes CLI.

Key facts:
- **CLI only** — does not work through Discord/gateway/WebUI. Must use `hermes` or `hermes chat` from terminal.
- Hermes auto-detects Brave, Chrome, Chromium, Edge at common install paths.
- Set permanent override: `hermes config set browser.cdp_url "http://127.0.0.1:9222"`
- Must use `--user-data-dir` (separate from normal profile) or the debug port won't come up if browser is already running.

### Path 2: `AGENT_BROWSER_HEADED=1` — Env Var

Add to `~/.hermes/.env`:
```bash
AGENT_BROWSER_HEADED=1
```

Supported by `agent-browser` CLI (vercel-labs/agent-browser). Passes through to Chrome to disable headless mode. Works through all Hermes modes including gateway/Discord.

### Other relevant env vars (local browser mode)

```bash
AGENT_BROWSER_ARGS=--no-sandbox              # Extra Chromium flags
BROWSER_INACTIVITY_TIMEOUT=120               # Auto-cleanup timeout (seconds)
```

Note: Hermes auto-injects `--no-sandbox,--disable-dev-shm-usage` when it detects root or restricted user namespaces. Setting `AGENT_BROWSER_ARGS` manually **disables** this auto-injection.

## Source: agent-browser GitHub

The `--headed` flag and `AGENT_BROWSER_HEADED` env var come from the `agent-browser` CLI (https://github.com/vercel-labs/agent-browser), which Hermes uses for local browser mode. These are not documented in Hermes docs directly — discovered via GitHub issue #470.

## Decision Matrix

| Scenario | Use |
|----------|-----|
| Interactive debug, see what agent does | `/browser connect` |
| Always visible, no CLI interaction | Config `browser.cdp_url` + pre-launched browser |
| Gateway mode (Discord/Telegram) | `AGENT_BROWSER_HEADED=1` |
| Cloud providers (Browserbase/Firecrawl) | Not applicable — cloud browsers are always remote |
