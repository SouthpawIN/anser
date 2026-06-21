# Terminal Venv Activation — Diagnosing Missing Packages on Debian Trixie

> Quick reference for when skills fail with "package not found" despite packages being installed in the Hermes venv.

## Root Cause

`terminal()` spawns a fresh `bash -l` login shell that does **not** have `VIRTUAL_ENV` set or the venv's `bin/` in the search PATH. System Python (e.g. `/usr/bin/python3` → 3.12) resolves instead of the venv's Python (3.11 via uv). On Debian Trixie, PEP 668 blocks `pip install` system-wide, so there's no fallback — the venv is the only place packages live.

## One-Command Fix

```bash
hermes config set terminal.shell_init_files '["~/.hermes/hermes-agent/venv/bin/activate"]'
```

Then restart Hermes. This sources the venv into every terminal session before any command runs.

**Important:** When `shell_init_files` is set, the default `~/.bashrc` auto-source is removed. Include it explicitly if needed:

```bash
hermes config set terminal.shell_init_files '["~/.bashrc","~/.hermes/hermes-agent/venv/bin/activate"]'
```

## Alternative Approaches

1. **`env_passthrough`** — pass VIRTUAL_ENV and PATH from parent shell:
   ```yaml
   terminal:
     env_passthrough: ["VIRTUAL_ENV", "PATH"]
   ```
   Less reliable across restarts.

2. **Add venv to PATH in `~/.bashrc`** — works because `auto_source_bashrc: true` is default:
   ```bash
   export PATH="$HOME/.hermes/hermes-agent/venv/bin:$PATH"
   ```

3. **`agent.environment_hint`** — tell the model to activate explicitly (less reliable):
   ```yaml
   agent:
     environment_hint: "The Hermes venv is at ~/.hermes/hermes-agent/venv. Always source it before running Python."
   ```

## Why `execute_code` Already Works

`code_execution.mode: project` (default) auto-detects `VIRTUAL_ENV` / `CONDA_PREFIX` and uses it as the Python interpreter. The gap is only with the `terminal()` tool.

## Diagnostic Commands

```bash
echo "$VIRTUAL_ENV"                              # Should show venv path
which python3                                      # Should resolve to venv bin
grep shell_init_files ~/.hermes/config.yaml        # Should be non-empty
ls ~/.hermes/hermes-agent/venv/bin/python3         # Verify venv exists
```

## Docs Reference

- FAQ: https://hermes-agent.nousresearch.com/docs/reference/faq (search "shell_init_files")
- Code Execution: https://hermes-agent.nousresearch.com/docs/user-guide/features/code-execution
- Configuration: https://hermes-agent.nousresearch.com/docs/user-guide/configuration
