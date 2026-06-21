---
name: hermes-docs-navigation
description: Use when linking to or citing Hermes Agent documentation (hermes-agent.nousresearch.com/docs). The Docusaurus site uses client-side routing — direct URL entry often 404s. Never guess URLs; always navigate through the live site and verify. Also covers the mandatory TL;DR + MEDIA response format and known-good URL list.
version: 1.2.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes-agent, docs, docusaurus, url-verification, response-format]
    related_skills: [hermes-agent-skill-authoring]
---

# Hermes Agent Documentation — Navigation & URL Verification

## Overview

The Hermes Agent docs site (`https://hermes-agent.nousresearch.com/docs`) is built on Docusaurus. Client-side routing means you CANNOT guess URL slugs — direct browser navigation to unvisited URLs returns 404 even if the page exists in the sidebar. The user will call you out publicly if you present fake links, so every URL must be verified through the live site.

## When to Use

- User asks for a guide/tutorial referencing Hermes docs
- You need to include an "official docs" link in a response
- User explicitly says "give me real links"
- You're writing a response that references any `/docs/` page on `hermes-agent.nousresearch.com`

**Don't use for:** Non-Hermes URLs (regular sites work fine with direct navigation). Don't use for the top-level `/docs` page itself (it always resolves).

## How to Verify a Docs URL

### Step-by-step

1. **Navigate to the docs root:**
   ```
   browser_navigate(url='https://hermes-agent.nousresearch.com/docs')
   ```

2. **Use the search box** (fastest for specific pages):
   - `browser_type` into the search box (`textbox "Search"` in the top nav)
   - `browser_press(key='Enter')` 
   - Wait for results — Docusaurus will navigate to the best match
   - Read `window.location.href` via `browser_console(expression='window.location.href')`

3. **Or browse the sidebar** (for exploration):
   - Click top-level section buttons to expand (e.g., "Using Hermes", "Features", "Reference")
   - Click the nested page link
   - Read `window.location.href` via browser console

4. **Record the verified URL** before using it in a response.

### Known URL pattern (Docusaurus v3)

URLs generally follow the pattern:
```
https://hermes-agent.nousresearch.com/docs/<section>/<page-slug>
```

**But the section nesting in the sidebar does NOT match the URL path.** For example:
- Sidebar: Reference → Command Reference → Profile Commands Reference
- Actual URL: `/docs/reference/profile-commands` (NOT `/docs/reference/command-reference/profile-commands-reference`)

This is why verification is mandatory — the sidebar breadcrumb path and the URL slug are independent in Docusaurus.

### Cross-check with web_extract

If `web_extract` is working (requires Firecrawl), you can use it as a secondary check:
```python
web_extract(urls=['https://hermes-agent.nousresearch.com/docs/reference/profile-commands'])
```
If it returns content (not 404), the URL is valid. If it 404s, try the browser navigation method.

## Verified URL Reference

See `references/verified-urls.md` for a list of confirmed URLs and known-404 URLs. Re-verify URLs before using them — Docusaurus slugs can change between versions.

Additional references:
- `references/telegram-draft-streaming-flicker.md` — Telegram `sendMessageDraft` flicker bug: symptoms, root cause, and config fix
- `references/browser-non-headless-modes.md` — running browser in visible mode via CDP attach or `AGENT_BROWSER_HEADED=1`
- `references/terminal-venv-activation.md` — skills failing with "package not found" on Debian Trixie: ensure `terminal()` finds the Hermes venv via `shell_init_files`
- `references/subagent-delegation-patterns.md` — subagent skills/model routing patterns: why skills don't inherit, how to embed in context, toolset mechanics from source
- `references/profile-distributions-gitignore.md` — profile distributions: why `git add .` adds everything despite docs saying files are "excluded"; the USER_OWNED_EXCLUDE frozenset is Hermes-internal, not a gitignore
| Page | Verified URL |
|------|-------------|
| Configuration | `/docs/user-guide/configuration` |
| Configuring Models | `/docs/user-guide/configuring-models` |
| Profiles: Running Multiple Agents | `/docs/user-guide/profiles` |
| Running Many Gateways at Once | `/docs/user-guide/multi-profile-gateways` |
| Web Dashboard | `/docs/user-guide/features/web-dashboard` |
| Desktop App | `/docs/user-guide/desktop` |
| Profile Distributions | `/docs/user-guide/profile-distributions` |
| Subagent Delegation | `/docs/user-guide/features/delegation` |
| Discord Setup | `/docs/user-guide/messaging/discord` |
| Profile Commands Reference | `/docs/reference/profile-commands` |
| Messaging Gateway (general) | `/docs/user-guide/messaging/` |
| Code Execution | `/docs/user-guide/features/code-execution` |
| FAQ (Troubleshooting) | `/docs/reference/faq` |
| Security (Dangerous Command Approval) | `/docs/user-guide/security` |
| TUI (Terminal UI) | `/docs/user-guide/tui` |
| CLI Interface | `/docs/user-guide/cli` |
| Sessions | `/docs/user-guide/sessions` |
| CLI Commands (hermes insights) | `/docs/reference/cli-commands#hermes-insights` |
| Checkpoints & Rollback | `/docs/user-guide/checkpoints-and-rollback` |
| OAuth over SSH | `/docs/guides/oauth-over-ssh` |
| FAQ (Multi-Model Workflows) | `/docs/reference/faq#using-different-models-for-different-tasks-multi-model-workflows` |

## Common Pitfalls

1. **Guessing URL slugs from sidebar labels.** The sidebar shows "Reference → Command Reference → Profile Commands Reference" but the URL is `/docs/reference/profile-commands`. Always verify.

2. **Direct `browser_navigate` to an unvisited Docusaurus page.** Docusaurus serves a 404 for any URL not previously navigated to through the app. Use the search box or sidebar navigation instead.

3. **Trusting `web_extract` alone.** When Firecrawl is down (as happened in this session), `web_extract` returns errors even for valid URLs. Fall back to browser navigation.

4. **Using hardcoded paths from old sessions.** Docusaurus URL slugs can change between versions. Re-verify if documentation references are stale.

5. **Linking to `/docs/features/management/web-dashboard` because the sidebar shows `Features → Management → Web Dashboard`.** The actual URL is `/docs/user-guide/features/web-dashboard`. The Docusaurus sidebar structure is a curated navigation tree, not a filesystem mirror.

6. **Page body goes blank after scrolling.** After repeated `browser_scroll` calls, `document.body.innerText` can report length 0 even though the snapshot shows content. Docusaurus may unload off-screen content. Re-navigate to the page to get a fresh load, or use `document.querySelector('article').innerText` instead of `document.body.innerText` to read content from the article container.

7. **Skills fail with 'package not found' on Debian (PEP 668).** When a skill runs `terminal("python script.py")` and the package IS installed but not found, the terminal shell likely isn't using the Hermes venv. The fix is `terminal.shell_init_files: ["~/.hermes/hermes-agent/venv/bin/activate"]` in config.yaml. See `references/terminal-venv-activation.md` for the full diagnostic and fix.

8. **Docusaurus search box is unreliable for targeted lookups.** Repeated searches with different query terms can land on the same internal/developer page multiple times. When search fails to find the right page, fall back to sidebar navigation: expand the relevant section (e.g., "Reference" → "Configuration Reference") and click the link directly. The sidebar always reflects the true page tree.

9. **Multi-word Docusaurus slugs use hyphens between ALL words, including "and".** The sidebar shows "Checkpoints & Rollback" but the URL is `/docs/.../checkpoints-and-rollback` — NOT `checkpoints-rollback` (missing "and") or `checkpoints--rollback` (double hyphen). When guessing a slug from a sidebar label, include EVERY word: "Checkpoints and Rollback" → `checkpoints-and-rollback`. But still verify — sidebar labels don't always map 1:1 to URL slugs.

10. **Docusaurus sidebar clicks may not navigate on the first attempt.** Client-side routing can silently fail when clicking sidebar links — the URL stays on the previous page. Always follow a sidebar click with `browser_console(expression='window.location.href')` to confirm you actually landed. If the URL didn't change, use the search box instead (type the page title and select from dropdown).

11. **Desktop App page slug is `/desktop`, NOT `/desktop-app`.** The sidebar says "Desktop App" but the URL is `/docs/user-guide/desktop`. Navigating to `/docs/user-guide/desktop-app` returns 404. Re-verify when linking.

12. **Profile distributions: "excluded from distributions" ≠ gitignore.** The docs say certain files are "excluded from distributions" but this refers to Hermes's internal `USER_OWNED_EXCLUDE` frozenset in `profile_distribution.py`, NOT a `.gitignore` file. Users running `git add .` on a used profile will accidentally commit `memories/`, `sessions/`, `state.db`, `.env`, etc. When answering profile distribution questions, always tell users to manually create a `.gitignore`. See `references/profile-distributions-gitignore.md` for the full diagnostic and ready-to-paste `.gitignore`.

## Extracting Truncated Content with `browser_console`

When `browser_snapshot` shows `[... N more lines truncated, use browser_snapshot for full content]`, the full page content is in the DOM but not in the snapshot. Use `browser_console` with an IIFE to extract it:

### Extract a specific section by heading text

```js
// Find content starting from a heading and return N characters
(() => {
  let t = document.querySelector('article').innerText;
  let idx = t.indexOf('Local Chromium-family browser via CDP');
  if (idx === -1) return 'NOT FOUND';
  return t.substring(idx, idx + 4000);
})()
```

### List all h2/h3 headings for navigation

```js
(() => {
  let headings = [...document.querySelectorAll('article h2, article h3')]
    .map(h => h.innerText);
  return headings.join('\n');
})()
```

### Search for a term and return surrounding context

```js
(() => {
  let t = document.querySelector('article').innerText;
  let idx = t.indexOf('headless');
  if (idx === -1) return 'NOT FOUND';
  return t.substring(Math.max(0, idx - 100), idx + 500);
})()
```

**Key patterns:**
- Always use IIFEs `(() => { ... })()` — variables persist across `browser_console` calls, so IIFEs prevent `Identifier has already been declared` errors
- Prefer `document.querySelector('article').innerText` over `document.body.innerText` — the article container stays populated even when the body is flushed
- Use `.substring(idx, idx + N)` to cap output; 3000-5000 chars is a good default
- Use `.indexOf()` searches on headings (e.g., `'Plugin Hooks'`, `'Shell Hooks'`) to jump to sections
- If `indexOf` returns `-1`, the section may not be in the DOM yet — scroll down and retry

## Reference Files

- `references/verified-urls.md` — confirmed-working and known-404 doc URLs
- `references/browser-non-headless-modes.md` — running Hermes browser in visible mode via CDP or `AGENT_BROWSER_HEADED=1`
- `references/telegram-draft-streaming-flicker.md` — Telegram `sendMessageDraft` flicker bug: symptoms, root cause, and config fix
- `references/terminal-venv-activation.md` — skills failing with "package not found" on Debian Trixie: venv activation via `shell_init_files`
- `references/local-provider-config.md` — configuring Ollama, vLLM, LM Studio, and other OpenAI-compatible local endpoints via `custom_providers` and `provider: custom`
- `references/discord-30032-max-commands.md` — Discord error 30032 (100-command limit): symptoms, root cause, and the `DISCORD_COMMAND_SYNC_POLICY` fix
- `references/lmstudio-show-chat-confusion.md` — LM Studio "Show Chat" button confusion: architecture diagram distinguishing LM Studio's built-in chat from Local Server API mode
- `references/stale-tool-result-branch-recovery.md` — recovering from stuck conversations via `/branch`, `/rewind`, and `/rollback` when a tool result can't resolve to its tool request ID
- `references/session-recovery-commands.md` — definitive guide to `/undo`, `/branch`, `/rewind`, `/rollback`: which to use when, how each works under the hood, and the common `/undo`-vs-`/branch` confusion

## Verification Checklist

- [ ] Navigated to the page through the Docusaurus site (search or sidebar)
- [ ] Read `window.location.href` from browser console to get the real URL
- [ ] URL is included in the response, not guessed
- [ ] URL is full `https://hermes-agent.nousresearch.com/docs/...` form
