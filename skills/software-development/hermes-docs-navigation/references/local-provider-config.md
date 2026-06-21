# Configuring Local LLM Providers (Ollama, vLLM, LM Studio, etc.)

> Condensed from Hermes source code and docs. Covers the two approaches
> for wiring any OpenAI-compatible local endpoint into Hermes Agent.
> 
> **See also:** `references/lmstudio-show-chat-confusion.md` for the common
> "Show Chat" button confusion — users often mistake LM Studio's built-in
> chat for the API server Hermes needs.

---

## Two Approaches

### Approach A: Named `custom_providers` (Recommended for Multiple Endpoints)

```yaml
# ~/.hermes/config.yaml
model:
  provider: "Ollama"          # matches a name in custom_providers
  model: qwen3-coder

custom_providers:
  - name: "Ollama"
    base_url: http://192.168.1.100:11434/v1
    api_key: ollama

  - name: "VLLM-Server"
    base_url: http://localhost:8000/v1
    api_key: not-needed
```

**Switch at runtime:** `/model Ollama qwen3-coder` (gateway) or `hermes model` (CLI).

**Key detail from docs:** The `provider` field references the `name` from
`custom_providers` directly — e.g., `provider: "Ollama"` works. You do NOT
set `provider: custom` when using a named entry (that would fall through to
the generic `provider: custom` path with env vars instead).

### Approach B: Generic `provider: custom` with Environment Variables

```yaml
# ~/.hermes/config.yaml
model:
  provider: custom
  model: qwen3-coder
```

```bash
# ~/.hermes/.env
OPENAI_BASE_URL=http://192.168.1.100:11434/v1
OPENAI_API_KEY=ollama
```

Or via CLI:
```bash
hermes config set OPENAI_BASE_URL http://192.168.1.100:11434/v1
hermes config set OPENAI_API_KEY ollama
hermes config set provider custom
hermes config set model qwen3-coder
```

**Limitation:** Only ONE custom endpoint at a time (env vars are global).

---

## Do NOT Use `provider: openai` for Local Endpoints

```yaml
# DANGER: semantically wrong, credential scoping issues
model:
  provider: openai           # this means "OpenAI, the company"
  model: qwen3-coder
```

**Why this is risky:**
- `provider: openai` is intended for the actual OpenAI API
- If you ever add a real `OPENAI_API_KEY`, Hermes will send it to your
  local Ollama/vLLM endpoint — credential leakage
- Hermes' credential scoping (from `OPENAI_BASE_URL`) may route real
  OpenAI keys to local endpoints and vice versa
- Use `provider: custom` or named `custom_providers` instead

---

## Ollama-Specific Notes

Ollama exposes an OpenAI-compatible API at `http://<host>:11434/v1`.
The `/v1` suffix is REQUIRED — without it, Hermes will 404.

Default API key: `ollama` (Ollama doesn't validate it, but Hermes requires
the field to be non-empty).

```bash
# Verify Ollama is serving correctly:
curl http://192.168.x.y:11434/v1/models
```

## vLLM / SGLang / text-generation-webui

Same pattern — any server that speaks the OpenAI `/v1/chat/completions`
protocol works:

```yaml
custom_providers:
  - name: "VLLM"
    base_url: http://localhost:8000/v1
    api_key: not-needed
```

Some servers default to `/v1` already; others need it explicitly.

## LM Studio

LM Studio is a **first-class provider** in Hermes (`provider: "lmstudio"`).
It defaults to `http://127.0.0.1:1234/v1`.

**Built-in approach (simplest):**
```bash
hermes config set provider lmstudio
hermes config set model <model-name>
```

Or with custom base URL/auth in `.env`:
```bash
LM_BASE_URL=http://192.168.1.50:1234/v1
LM_API_KEY=lm-studio
```

**Critical distinction:** LM Studio's "Chat" tab is its own built-in chat UI —
NOT used with Hermes. You must use the **"Local Server"** tab and click "Start
Server" to expose the API that Hermes connects to. See
`references/lmstudio-show-chat-confusion.md` for the full architecture diagram
and user-facing explanation.

Hermes can auto-detect the model name and context length from LM Studio's
native API (`/api/v1/models` endpoint).

---

## Verification

```bash
hermes status
# Should show:
#   Provider: Ollama (custom)
#   Model: qwen3-coder
#   Base URL: http://192.168.x.y:11434/v1
```

---

## Source References

- `gateway/platforms/base.py` — `MessageEvent.auto_skill` field, provider resolution
- `hermes_cli/runtime_provider.py` — `_resolve_custom_runtime()` for `custom_providers`
- `plugins/model-providers/` — bundled provider plugins that declare `api_mode`, `base_url`, `env_vars`
- Docs: https://hermes-agent.nousresearch.com/docs/user-guide/configuration
- Docs: https://hermes-agent.nousresearch.com/docs/user-guide/configuring-models
