---
name: termux-tts-voice-fix
description: Fix silent TTS on Android/Termux when TTS files generate but audio does not play through speakers. Diagnoses whether the issue is in the playback fallback chain (voice_mode.py) or the missing auto-play trigger (tts_tool.py), patches the correct layer, and covers launching Android apps via termux-open.
version: 2.0.0
author: Hermes Agent
metadata:
  tags: [termux, android, tts, voice, audio, mpv, termux-media-player, troubleshooting, termux-open]
---

# Fix Silent TTS on Termux/Android

On Android/Termux, the `text_to_speech` tool may appear to work (files generate in `~/.hermes/audio_cache/`) but you hear **no audio**. There are **two separate code layers** that can cause this, and you must identify which one is broken before patching.

> **Critical discovery (v2.0):** Hermes has TWO audio playback paths:
> 1. **`voice_mode.py`** — used by the CLI's `/voice tts` toggle. It calls `play_audio_file()` which has a fallback chain of system audio players (`afplay`, `ffplay`, `mpv`, `aplay`).
> 2. **`tts_tool.py`** — the `text_to_speech_tool()` itself. It **generates audio but does NOT call `play_audio_file()`**. It simply returns a `MEDIA:` tag string and expects a messaging platform (Telegram, Discord, etc.) to deliver it.
>
> On Termux there is no messaging platform to intercept the `MEDIA:` tag, so the audio file sits on disk silently unless `tts_tool.py` is patched to trigger local playback.

## Quick Diagnosis (Do This First)

Before patching anything, determine which layer is failing:

```bash
# Test 1: Can mpv play an existing file at all?
timeout 10 mpv --no-video --really-quiet ~/.hermes/audio_cache/*.mp3 && echo "mpv works" || echo "mpv broken or missing"

# Test 2: Is the voice_mode.py mpv entry already present?
grep -n 'players.append.*mpv' ~/.hermes/hermes-agent/tools/voice_mode.py

# Test 3: Does tts_tool.py call play_audio_file?
grep -n 'play_audio_file' ~/.hermes/hermes-agent/tools/tts_tool.py
```

**Interpretation:**
- If Test 1 fails → install mpv: `pkg install mpv`
- If Test 2 shows no results → apply **Layer 1** patch (`voice_mode.py`)
- If Test 3 shows results ONLY in the `_play_via_tempfile` fallback (internal streaming) and NOT in the main `text_to_speech_tool()` return path → apply **Layer 2** patch (`tts_tool.py`)
- If Test 2 AND Test 3 both look okay but you still hear nothing → check `TERMUX_VERSION`/`PREFIX` env detection or pycache staleness

## Symptoms by Layer

### Layer 1 — voice_mode.py broken
- `/voice tts` toggles without error
- CLI voice mode shows "TTS on"
- `play_audio_file()` may return `True` but you hear nothing
- `grep` shows NO `mpv` entry in `voice_mode.py`

### Layer 2 — tts_tool.py missing auto-play
- TTS files are created successfully in `~/.hermes/audio_cache/`
- `mpv` works when you test manually
- `voice_mode.py` already contains the mpv fallback
- The tool returns `MEDIA:/path/to/file` but no playback happens
- This affects the `text_to_speech` tool directly, NOT just `/voice tts`

## Fix

### Layer 1: Add `mpv` to Hermes fallback chain (voice_mode.py)

Only needed if `voice_mode.py` does not already list `mpv`.

**Step 1: Locate the file**

```bash
VOICE_MODE="${HERMES_HOME:-$HOME/.hermes}/hermes-agent/tools/voice_mode.py"
```

**Step 2: Patch the player list**

Find the `players = []` block inside `play_audio_file()` (~line 885). Ensure the `mpv` entry exists:

```python
    if system == "Darwin":
        players.append(["afplay", file_path])
    players.append(["ffplay", "-nodisp", "-autoexit", "-loglevel", "quiet", file_path])
    players.append(["mpv", "--no-video", "--really-quiet", file_path])  # <-- MUST EXIST
    if system == "Linux":
        players.append(["aplay", "-q", file_path])
```

**Step 3: Clear the bytecode cache**

```bash
rm -f "${HERMES_HOME:-$HOME/.hermes}/hermes-agent/tools/__pycache__/voice_mode.cpython-313.pyc"
```

(Adjust `cpython-313` to your Python version.)

---

### Layer 2: Patch tts_tool.py to auto-play locally on Termux

This is the **most common root cause** on Android/Termux. The `text_to_speech_tool()` generates audio but returns it as a `MEDIA:` tag for messaging platforms. On Termux there is no messaging platform, so nobody plays the file.

**Step 1: Locate the file**

```bash
TTS_TOOL="${HERMES_HOME:-$HOME/.hermes}/hermes-agent/tools/tts_tool.py"
```

**Step 2: Find the return block**

Inside `text_to_speech_tool()`, locate the block near the end where the JSON result is built. It looks like:

```python
        file_size = os.path.getsize(file_str)
        logger.info("TTS audio saved: %s (%s bytes, provider: %s)", file_str, f"{file_size:,}", provider)

        # Build response with MEDIA tag for platform delivery
        media_tag = f"MEDIA:{file_str}"
```

**Step 3: Insert the auto-play trigger**

Add this block immediately after the `logger.info` line and **before** the `media_tag` assignment:

```python
        # Auto-play locally on Android/Termux where no messaging platform handles MEDIA tags
        if os.environ.get("TERMUX_VERSION") or os.environ.get("PREFIX", "").startswith("/data/data/com.termux"):
            try:
                from tools.voice_mode import play_audio_file
                play_audio_file(file_str)
            except Exception as exc:
                logger.debug("Local TTS playback skipped on Termux: %s", exc)
```

**Step 4: Clear the bytecode cache**

```bash
rm -f "${HERMES_HOME:-$HOME/.hermes}/hermes-agent/tools/__pycache__/tts_tool.cpython-313.pyc"
```

**Step 5: Restart Hermes**

Python caches imported modules in memory. The patch is on disk, but the running session still holds the old code. You **must** start a new session for the patch to take effect:

```bash
# Inside Hermes CLI
/reset
```

Or exit and relaunch entirely.

---

### Layer 3: Ensure `mpv` actually produces audible output

On some Android devices, `mpv` routes through Linux audio subsystems (PulseAudio/ALSA) that are not wired to the device's speakers. Test directly:

```bash
timeout 10 mpv --no-video --really-quiet ~/.hermes/audio_cache/*.mp3
```

If `mpv` returns 0 but you hear nothing, use **`termux-media-player`** instead.

**Requirements:**
- `termux-api` package: `pkg install termux-api`
- **Termux:API Android app** installed from F-Droid or GitHub releases

**Test:**
```bash
termux-media-player play ~/.hermes/audio_cache/tts_test.mp3
termux-media-player info   # Shows "Status: Playing"
```

If you need to swap `mpv` for `termux-media-player` inside the Hermes fallback chain, edit `voice_mode.py` and replace the `mpv` entry with:

```python
    players.append(["termux-media-player", "play", file_path])
```

> **Warning:** `termux-media-player` blocks during playback and is less suited for the CLI voice loop than `mpv`. It is best used as a fallback only when `mpv` is silent.

## Prerequisites

- `mpv` installed: `pkg install mpv` (if missing)
- `termux-api` installed: `pkg install termux-api` (for termux-media-player fallback)
- **Termux:API Android app** installed (F-Droid or GitHub releases)
- Edge TTS configured as provider: `hermes config set tts.provider edge`
- TTS toolset enabled: `hermes tools enable tts`

## Pitfalls

- **Two layers, two patches.** The most common mistake is patching `voice_mode.py` and assuming that fixes everything. If the `text_to_speech` tool is silent, the real fix is in `tts_tool.py`.
- **Pycache must be cleared.** After editing `voice_mode.py` or `tts_tool.py`, delete the corresponding `__pycache__/*.pyc` file. Python will not re-read the source until the cache is stale or removed.
- **Session restart required.** Even after clearing pycache, a running Hermes process has already imported the old module into memory. Use `/reset` or exit and relaunch.
- **`tts.auto_tts` vs `voice.auto_tts` mismatch.** The stock CLI startup code only reads `voice.auto_tts`. If you want TTS on by default, set `voice.auto_tts` (not just `tts.auto_tts`):
  ```bash
  hermes config set voice.auto_tts true
  ```
- **`mpv` may return `True` but stay silent.** On Android, `mpv` can appear to succeed while producing no audible output. Always verify with your ears, not just the return value.
- **`termux-media-player` blocks during playback.** Unlike `mpv --really-quiet`, it blocks until the track finishes. In the CLI's `voice_mode.py`, the `mpv` fallback is preferred because it doesn't block the UI thread.
- **Process cleanup is required.** If `termux-media-player` is interrupted, stale processes can hold the audio lock. Kill old instances: `pkill -f "termux-media-player"`
- If you update Hermes (`hermes update`), you may need to re-apply these patches.

## Auto-Play Watcher (Persistent Voice Mode)

The `/voice tts` toggle only works within a single Hermes CLI session and still requires the CLI layer to invoke `play_audio_file()`. If you want **every voice file to play automatically** — even when chatting via tools, gateway, or API — run a background watcher.

### Why Use a Watcher

- Works across sessions (no need to type `/voice tts` every time)
- Catches voice files generated by `text_to_speech` tool calls, not just CLI voice mode
- Plays instantly (within ~500ms of file creation)
- Persists in a `tmux` session even if the main Hermes process restarts

### Setup

**1. Create the watcher script** at `~/.hermes/auto_play_tts.py`:

```python
#!/data/data/com.termux/files/usr/bin/env python3
import os, time, subprocess, glob

HERMES_HOME = os.path.expanduser("~/.hermes")
AUDIO_DIRS = [os.path.join(HERMES_HOME, "audio_cache"), HERMES_HOME]
SEEN = set()

def find_audio():
    files = []
    for d in AUDIO_DIRS:
        if os.path.isdir(d):
            files.extend(glob.glob(os.path.join(d, "*.mp3")))
            files.extend(glob.glob(os.path.join(d, "*.ogg")))
    return sorted(files, key=os.path.getmtime)

# Seed existing files
for f in find_audio():
    SEEN.add(f)

while True:
    for f in find_audio():
        if f not in SEEN:
            SEEN.add(f)
            subprocess.run(
                ["mpv", "--no-video", "--really-quiet", f],
                stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL,
                timeout=300, check=False,
            )
    time.sleep(0.5)
```

**2. Start it in a persistent tmux session:**

```bash
chmod +x ~/.hermes/auto_play_tts.py
tmux new-session -d -s tts_watcher "python3 ~/.hermes/auto_play_tts.py"
```

**3. Verify it's running:**

```bash
tmux ls | grep tts_watcher
ps aux | grep auto_play_tts | grep -v grep
```

**4. Stop / restart:**

```bash
tmux kill-session -t tts_watcher        # stop
tmux new-session -d -s tts_watcher "python3 ~/.hermes/auto_play_tts.py"  # restart
```

### Integration with Gateway Platforms

On Telegram, Discord, WhatsApp, etc., the gateway already sends voice files natively. The watcher is mainly for the **Termux CLI** where no native auto-play exists.

## Related: Launching Android Apps from Termux

A common companion task on Android/Termux is opening apps or URLs from the CLI. Direct `am start` fails on modern Android with:

```
SecurityException: Permission Denial: package=com.android.shell does not belong to uid=...
```

### Workaround: `termux-open`

The `termux-tools` package provides `termux-open`, which sends a **broadcast intent** to the Termux app itself. Termux then handles the launch with proper Android permissions.

**Open a URL in the default app:**

```bash
termux-open "https://youtube.com"
```

**Force a specific app via intent:**

```bash
am broadcast --user 0 -a android.intent.action.VIEW \
  -n com.termux/com.termux.app.TermuxOpenReceiver \
  -d "https://youtube.com"
```

**Open a specific installed app by package name:**

First find the package:

```bash
pm list packages | grep youtube
```

Then use `termux-open` with a deep-link or URL scheme if the app supports it. Not all apps expose deep-links, but most handle `https://` URLs via their verified domain.

**Requirements:** `pkg install termux-tools` (usually pre-installed).
