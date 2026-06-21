---
name: smart-home-iot
description: "Smart home & IoT: Philips Hue lights (OpenHue CLI), TouchDesigner real-time visuals via twozero MCP."
version: 1.0.0
author: Hermes Agent
license: MIT
tags: ["smart-home", "iot", "hue", "lights", "touchdesigner", "mcp", "automation", "real-time-visuals"]
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [smart-home, iot, hue, lights, touchdesigner, mcp, automation, real-time-visuals]
    category: smart-home
    related_skills: [social-media-communication, productivity-office-tools, creative-coding-animation]
---

# Smart Home & IoT

Unified skill for smart home automation and real-time visual control. Covers two domains:

1. **Philips Hue** (`openhue`): Control lights, scenes, rooms via OpenHue CLI
2. **TouchDesigner** (`touchdesigner-mcp`): Real-time visuals via twozero MCP (36 native tools)

Load this skill when the user wants to:
- Control Philips Hue lights (on/off, brightness, color, temperature, scenes)
- Automate lighting with cron/schedules
- Build real-time visuals in TouchDesigner
- Create generative art, audio-reactive visuals, VJ sets
- Control TouchDesigner operators, parameters, wiring via MCP

---

## Quick Decision Guide

| Goal | Use This Section |
|------|------------------|
| Turn Hue lights on/off/dim | [Philips Hue (OpenHue)](#philips-hue-openhue) |
| Set Hue scenes (movie mode, bedtime) | [Philips Hue (OpenHue)](#philips-hue-openhue) |
| Automate lighting schedules | [Philips Hue (OpenHue)](#philips-hue-openhue) |
| Real-time generative visuals | [TouchDesigner (MCP)](#touchdesigner-mcp-twozero) |
| Audio-reactive GLSL shaders | [TouchDesigner (MCP)](#touchdesigner-mcp-twozero) |
| VJ / installation visuals | [TouchDesigner (MCP)](#touchdesigner-mcp-twozero) |
| MIDI/OSC control | [TouchDesigner (MCP)](#touchdesigner-mcp-twozero) |

---

## Philips Hue (OpenHue)

> **Source:** Absorbed from `openhue` skill. Control Philips Hue lights, scenes, rooms via OpenHue CLI.

### When to Use
- "Turn on/off the lights"
- "Dim the living room lights"
- "Set a scene" or "movie mode"
- Control specific Hue rooms, zones, or individual bulbs
- Adjust brightness, color, or color temperature

### Prerequisites
```bash
# Linux (pre-built binary)
curl -sL https://github.com/openhue/openhue-cli/releases/latest/download/openhue-linux-amd64 -o ~/.local/bin/openhue && chmod +x ~/.local/bin/openhue

# macOS
brew install openhue/cli/openhue-cli
```

**First run:** Press the button on your Hue Bridge to pair. Bridge must be on same local network.

### Common Commands

#### List Resources
```bash
openhue get light       # List all lights
openhue get room        # List all rooms
openhue get scene       # List all scenes
```

#### Control Lights
```bash
# Turn on/off
openhue set light "Bedroom Lamp" --on
openhue set light "Bedroom Lamp" --off

# Brightness (0-100)
openhue set light "Bedroom Lamp" --on --brightness 50

# Color temperature (warm to cool: 153-500 mirek)
openhue set light "Bedroom Lamp" --on --temperature 300

# Color (by name or hex)
openhue set light "Bedroom Lamp" --on --color red
openhue set light "Bedroom Lamp" --on --rgb "#FF5500"
```

#### Control Rooms
```bash
# Turn off entire room
openhue set room "Bedroom" --off

# Set room brightness
openhue set room "Bedroom" --on --brightness 30
```

#### Scenes
```bash
openhue set scene "Relax" --room "Bedroom"
openhue set scene "Concentrate" --room "Office"
```

### Quick Presets
```bash
# Bedtime (dim warm)
openhue set room "Bedroom" --on --brightness 20 --temperature 450

# Work mode (bright cool)
openhue set room "Office" --on --brightness 100 --temperature 250

# Movie mode (dim)
openhue set room "Living Room" --on --brightness 10

# Everything off
openhue set room "Bedroom" --off
openhue set room "Office" --off
openhue set room "Living Room" --off
```

### Notes
- Bridge must be on same local network
- First run requires pressing Hue Bridge button
- Colors only work on color-capable bulbs
- Names are case-sensitive — use `openhue get light` to check exact names
- Works great with cron jobs for scheduled lighting

---

## TouchDesigner (twozero MCP)

> **Source:** Absorbed from `touchdesigner-mcp` skill. Control running TouchDesigner via twozero MCP — create operators, set parameters, wire connections, execute Python, build real-time visuals. 36 native tools.

### Critical Rules
1. **NEVER guess parameter names** — Call `td_get_par_info` for the op type FIRST
2. **If `tdAttributeError` fires, STOP** — Call `td_get_operator_info` on failing node
3. **NEVER hardcode absolute paths** in script callbacks — Use `me.parent()` / `scriptOp.parent()`
4. **Prefer native MCP tools over `td_execute_python`** — Use `td_create_operator`, `td_set_operator_pars`, `td_get_errors`
5. **Call `td_get_hints` before building** — Returns patterns specific to op type

### Architecture
```
Hermes Agent -> MCP (Streamable HTTP) -> twozero.tox (port 40404) -> TD Python
```
- 36 native tools, free plugin (no payment/license)
- Context-aware (knows selected OP, current network)
- Hub health: `GET http://localhost:40404/mcp` returns JSON with PID, project name, TD version

### Setup (Automated)
```bash
bash "${HERMES_HOME:-$HOME/.hermes}/skills/creative/touchdesigner-mcp/scripts/setup.sh"
```
The script:
1. Checks if TD is running
2. Downloads twozero.tox if not cached
3. Adds `twozero_td` MCP server to Hermes config
4. Tests MCP connection on port 40404
5. Reports remaining manual steps

### Manual Steps (one-time)
1. **Drag `~/Downloads/twozero.tox` into TD network editor** → click Install
2. **Enable MCP:** twozero icon → Settings → mcp → "auto start MCP" → Yes
3. **Restart Hermes session** to pick up new MCP server

After setup, verify:
```bash
nc -z 127.0.0.1 40404 && echo "twozero MCP: READY"
```

### Environment Notes
- **Non-Commercial TD** caps resolution at 1280×1280 — use `outputresolution = 'custom'`
- **Codecs:** `prores` (preferred macOS) or `mjpa` fallback — H.264/H.265/AV1 need Commercial license
- **Always call `td_get_par_info` before setting params** — names vary by TD version

### Workflow

#### Step 0: Discover (before building)
```
Call td_get_par_info for each op type you plan to use
Call td_get_hints for your topic (e.g. "glsl", "audio reactive", "feedback")
Call td_get_focus to see what's selected
Call td_get_network to see what exists
```

#### Step 1: Clean + Build
**Split cleanup and creation into SEPARATE MCP calls** — destroying and recreating same-named nodes in one `td_execute_python` causes "Invalid OP object" errors.

```bash
# Create operators
td_create_operator type="noiseTOP" parent="/project1" name="bg" parameters={"resolutionw": 1280, "resolutionh": 720}
td_create_operator type="levelTOP" parent="/project1" name="brightness"
td_create_operator type="nullTOP" parent="/project1" name="out"

# For bulk creation/wiring, use td_execute_python
```

#### Step 2: Set Parameters
```bash
# Prefer native tool (validates params)
td_set_operator_pars path="/project1/bg" parameters={"roughness": 0.6, "monochrome": true}

# For expressions, use td_execute_python
```

#### Step 3: Wire (use td_execute_python)
```python
op('/project1/bg').outputConnectors[0].connect(op('/project1/fx').inputConnectors[0])
```

#### Step 4: Verify
```bash
td_get_errors path="/project1" recursive=true
td_get_perf
td_get_operator_info path="/project1/out" detail="full"
```

#### Step 5: Display / Capture
```bash
td_get_screenshot path="/project1/out"
```

### Key Implementation Rules

**GLSL time:** No `uTDCurrentTime` — use Values page:
```bash
td_set_operator_pars path="/project1/shader" parameters={"value0name": "uTime"}
# Then via script: op('/project1/shader').par.value0.expr = "absTime.seconds"
```

**Feedback TOP:** Use `top` parameter reference, not direct input wire. "Cook dependency loop" warning expected.

**Resolution:** Non-Commercial caps at 1280×1280 — use `outputresolution = 'custom'`.

**Large shaders:** Write GLSL to `/tmp/file.glsl`, then `td_write_dat` or `td_execute_python` to load.

**Vertex/Point access (TD 2025.32):** `point.P[0]`, `point.P[1]`, `point.P[2]` — NOT `.x`, `.y`, `.z`.

**Extensions:** `ext0object` = `"op('./datName').module.ClassName(me)"` in CONSTANT mode. After editing, call `td_reinit_extension`.

**Script callbacks:** ALWAYS use relative paths via `me.parent()` / `scriptOp.parent()`.

### Recording/Exporting Video
```python
# MovieFileOut TOP (NOT TOP.save())
rec = op('/project1').create(moviefileoutTOP, 'recorder')
op('/project1/out').outputConnectors[0].connect(rec.inputConnectors[0])
rec.par.type = 'movie'
rec.par.file = '/tmp/output.mov'
rec.par.videocodec = 'prores'  # NOT H.264 (needs Commercial license)
rec.par.record = True
```
Extract frames: `ffmpeg -i /tmp/output.mov -vframes 120 /tmp/frames/frame_%06d.png`

### Audio-Reactive GLSL (Proven Recipe)

#### Correct Signal Chain
```
AudioFileIn CHOP (playmode=sequential)
  → AudioSpectrum CHOP (FFT=512, outputmenu=setmanually, outlength=256, timeslice=ON)
  → Math CHOP (gain=10)
  → CHOP to TOP (dataformat=r, layout=rowscropped)
  → GLSL TOP input 1 (spectrum texture, 256x2)

Constant TOP (rgba32float, time) → GLSL TOP input 0
GLSL TOP → Null TOP → MovieFileOut
```

#### Critical Rules
1. **TimeSlice ON** for AudioSpectrum — OFF = 24000+ samples → overflow
2. **Set Output Length manually** to 256 via `outputmenu='setmanually'`
3. **NO Lag CHOP** for spectrum smoothing — expands 256 to 2400+, averages to near-zero
4. **NO Filter CHOP** either — same timeslice expansion problem
5. **Smoothing in GLSL** via temporal lerp: `mix(prevValue, newValue, 0.3)`
6. **CHOP to TOP dataformat = 'r'**, layout = 'rowscropped'
7. **Math gain = 10** (not 5) — raw spectrum ~0.19 in bass
8. **No Resample CHOP needed** — control size via AudioSpectrum `outlength`

#### GLSL Spectrum Sampling
```glsl
float iTime = texture(sTD2DInputs[0], vec2(0.5)).r;
float bass = (texture(sTD2DInputs[1], vec2(0.02, 0.25)).r + texture(sTD2DInputs[1], vec2(0.05, 0.25)).r) / 2.0;
float mid  = (texture(sTD2DInputs[1], vec2(0.2, 0.25)).r + texture(sTD2DInputs[1], vec2(0.35, 0.25)).r) / 2.0;
float hi   = (texture(sTD2DInputs[1], vec2(0.6, 0.25)).r + texture(sTD2DInputs[1], vec2(0.8, 0.25)).r) / 2.0;
```

### Operator Quick Reference
| Family | Color | Python class / MCP type | Suffix |
|--------|-------|------------------------|--------|
| TOP | Purple | noiseTOP, glslTOP, compositeTOP, levelTop, blurTOP, textTOP, nullTOP | TOP |
| CHOP | Green | audiofileinCHOP, audiospectrumCHOP, mathCHOP, lfoCHOP, constantCHOP | CHOP |
| SOP | Blue | gridSOP, sphereSOP, transformSOP, noiseSOP | SOP |
| DAT | White | textDAT, tableDAT, scriptDAT, webserverDAT | DAT |
| MAT | Yellow | phongMAT, pbrMAT, glslMAT, constMAT | MAT |
| COMP | Gray | geometryCOMP, containerCOMP, cameraCOMP, lightCOMP, windowCOMP | COMP |

### References (from original skill)
- `references/pitfalls.md` — Hard-won lessons
- `references/operators.md` — All operator families
- `references/network-patterns.md` — Audio-reactive, generative, GLSL, instancing recipes
- `references/mcp-tools.md` — Full 36-tool reference
- `references/python-api.md` — TD Python: op(), scripting, extensions
- `references/troubleshooting.md` — Connection diagnostics
- `references/audio-reactive.md` — Audio band extraction, beat detection
- `references/animation.md` — LFOs, timers, keyframes, easing

---

## Decision Matrix

| Task | Tool | Why |
|------|------|-----|
| Light control, scenes, schedules | OpenHue | Native Hue Bridge CLI, local network |
| Scheduled lighting automation | OpenHue + cron | Works great with `hermes cron add` |
| Real-time generative visuals | TouchDesigner MCP | 36 native tools, context-aware |
| Audio-reactive shaders | TouchDesigner MCP | Proven signal chain, GLSL access |
| VJ / live visuals | TouchDesigner MCP | Real-time, MIDI/OSC, projection mapping |
| Installation art | TouchDesigner MCP | Multi-window, projection mapping, DMX |

---

## Related Skills

- **`social-media-communication`** — Share creations on X/Yuanbao
- **`productivity-office-tools`** — Calendar integration for lighting schedules
- **`creative-coding-animation`** — Manim, p5js, ASCII video for complementary outputs
- **`creative-generative-ai`** — ComfyUI for AI-generated textures/assets
