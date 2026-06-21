---
name: creative-ascii
description: "ASCII art & animation: text banners (pyfiglet, FIGlet), cowsay, boxes, image-to-ASCII, and ASCII video conversion (video/audio to colored ASCII MP4/GIF)."
version: 1.0.0
author: Hermes Agent
license: MIT
tags: ["creative", "ascii", "art", "animation", "video", "text-art", "pyfiglet", "cowsay", "boxes", "image-to-ascii"]
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [creative, ascii, art, animation, video, text-art, pyfiglet, cowsay, boxes, image-to-ascii]
    category: creative
    related_skills: [creative-diagrams-visualizations, creative-generative-ai, creative-coding-animation, creative-design-systems]
---

# ASCII Art & Animation

Unified skill for text-based visual art: static ASCII art (banners, characters, boxes, image conversion) and animated ASCII video (video/audio to colored ASCII MP4/GIF).

**Contents:**
1. **ASCII Art** — Text banners, cowsay, boxes, image-to-ASCII, pre-made art search (ascii-art)
2. **ASCII Video** — Convert video/audio to colored ASCII MP4/GIF with effects pipeline (ascii-video)

Load this skill when the user wants to:
- Create text banners (project titles, headers, logos)
- Wrap messages in fun character art (cowsay)
- Add decorative borders/frames (boxes)
- Convert images to ASCII art
- Search for pre-made ASCII art (cats, rockets, dragons, etc.)
- Convert video/audio to animated ASCII art (MP4, GIF)
- Create audio-reactive ASCII visualizations
- Generate ASCII art QR codes, weather art

---

## Quick Decision Guide

| Goal | Use This Section |
|------|------------------|
| Text banner/logo | [ASCII Art → Text Banners](#text-banners-pyfiglet--local) |
| Fun message with character | [ASCII Art → Cowsay](#cowsay-message-art) |
| Decorative border/frame | [ASCII Art → Boxes](#boxes-decorative-borders) |
| Convert image to ASCII | [ASCII Art → Image to ASCII](#image-to-ascii-art) |
| Pre-made art (cat, rocket, dragon) | [ASCII Art → Search Pre-Made](#search-pre-made-ascii-art) |
| Video → ASCII animation | [ASCII Video](#ascii-video-production-pipeline) |
| Audio → ASCII visualization | [ASCII Video → Audio-Reactive Mode](#modes) |
| Generative ASCII animation | [ASCII Video → Generative Mode](#modes) |
| ASCII QR code / weather | [ASCII Art → Fun ASCII Utilities](#fun-ascii-utilities-via-curl) |

---

## ASCII Art

> **Source:** Absorbed from `ascii-art` skill. ASCII art: pyfiglet, cowsay, boxes, image-to-ascii.

### Tool 1: Text Banners (pyfiglet — Local)

Render text as large ASCII art banners. 571 built-in fonts.

#### Setup
```bash
pip install pyfiglet --break-system-packages -q
```

#### Usage
```bash
python3 -m pyfiglet "YOUR TEXT" -f slant
python3 -m pyfiglet "TEXT" -f doom -w 80       # Set width
python3 -m pyfiglet --list_fonts               # List all 571 fonts
```

#### Recommended Fonts
| Style | Font | Best For |
|-------|------|----------|
| Clean & modern | `slant` | Project names, headers |
| Bold & blocky | `doom` | Titles, logos |
| Big & readable | `big` | Banners |
| Classic banner | `banner3` | Wide displays |
| Compact | `small` | Subtitles |
| Cyberpunk | `cyberlarge` | Tech themes |
| 3D effect | `3-d` | Splash screens |
| Gothic | `gothic` | Dramatic text |

#### Tips
- Preview 2-3 fonts, let user pick
- Short text (1-8 chars) → detailed fonts (`doom`, `block`)
- Long text → compact fonts (`small`, `mini`)

### Tool 2: Text Banners (asciified API — Remote, No Install)

Free REST API, 250+ FIGlet fonts. Returns plain text directly.

```bash
# Basic
curl -s "https://asciified.thelicato.io/api/v2/ascii?text=Hello+World"

# Specific font
curl -s "https://asciified.thelicato.io/api/v2/ascii?text=Hello&font=Slant"
curl -s "https://asciified.thelicato.io/api/v2/ascii?text=Hello&font=Doom"
curl -s "https://asciified.thelicato.io/api/v2/ascii?text=Hello&font=Star+Wars"

# List fonts
curl -s "https://asciified.thelicato.io/api/v2/fonts"
```
- URL-encode spaces as `+`
- Response is plain text — ready to display
- Font names case-sensitive; use fonts endpoint for exact names

### Tool 3: Cowsay (Message Art)

Classic tool wrapping text in speech bubble with ASCII character.

#### Setup
```bash
sudo apt install cowsay -y    # Debian/Ubuntu
# brew install cowsay         # macOS
```

#### Usage
```bash
cowsay "Hello World"
cowsay -f tux "Linux rules"        # Tux penguin
cowsay -f dragon "Rawr!"           # Dragon
cowsay -f stegosaurus "Roar!"      # Stegosaurus
cowthink "Hmm..."                   # Thought bubble
cowsay -l                          # List all 50+ characters
```

#### Characters (50+)
`beavis.zen`, `bong`, `bunny`, `cheese`, `daemon`, `default`, `dragon`, `dragon-and-cow`, `elephant`, `eyes`, `flaming-skull`, `ghostbusters`, `hellokitty`, `kiss`, `kitty`, `koala`, `luke-koala`, `mech-and-cow`, `meow`, `moofasa`, `moose`, `ren`, `sheep`, `skeleton`, `small`, `stegosaurus`, `stimpy`, `supermilker`, `surgery`, `three-eyes`, `turkey`, `turtle`, `tux`, `udder`, `vader`, `vader-koala`, `www`

#### Eye/Tongue Modifiers
```bash
cowsay -b "Borg"       # =_=
cowsay -d "Dead"       # x_x
cowsay -g "Greedy"     # $_$
cowsay -p "Paranoid"   # @_@
cowsay -s "Stoned"     # *_*
cowsay -w "Wired"      # O_O
cowsay -e "OO" "Msg"   # Custom eyes
cowsay -T "U " "Msg"   # Custom tongue
```

### Tool 4: Boxes (Decorative Borders)

Draw decorative ASCII borders/frames around text. 70+ designs.

#### Setup
```bash
sudo apt install boxes -y    # Debian/Ubuntu
# brew install boxes         # macOS
```

#### Usage
```bash
echo "Hello World" | boxes                    # Default
echo "Hello World" | boxes -d stone           # Stone border
echo "Hello World" | boxes -d parchment       # Parchment scroll
echo "Hello World" | boxes -d cat             # Cat border
echo "Hello World" | boxes -d dog             # Dog border
echo "Hello World" | boxes -d unicornsay      # Unicorn
echo "Hello World" | boxes -d diamonds        # Diamond pattern
echo "Hello World" | boxes -d c-cmt           # C-style comment
echo "Hello World" | boxes -d html-cmt        # HTML comment
echo "Hello World" | boxes -a c               # Center text
boxes -l                                       # List all 70+ designs
```

#### Combine with pyfiglet/asciified
```bash
python3 -m pyfiglet "HERMES" -f slant | boxes -d stone
curl -s "https://asciified.thelicato.io/api/v2/ascii?text=HERMES&font=Slant" | boxes -d stone
```

### Tool 5: TOIlet (Colored Text Art)

Like pyfiglet with ANSI color effects and visual filters.

#### Setup
```bash
sudo apt install toilet toilet-fonts -y    # Debian/Ubuntu
# brew install toilet                      # macOS
```

#### Usage
```bash
toilet "Hello World"                       # Basic
toilet -f bigmono12 "Hello"                # Specific font
toilet --gay "Rainbow!"                    # Rainbow coloring
toilet --metal "Metal!"                    # Metallic effect
toilet -F border "Bordered"                # Add border
toilet -F border --gay "Fancy!"            # Combined
toilet -f pagga "Block"                    # Block-style (unique)
toilet -F list                             # List filters
```

#### Filters
`crop`, `gay` (rainbow), `metal`, `flip`, `flop`, `180`, `left`, `right`, `border`

> **Note:** Outputs ANSI escape codes — works in terminals, may not render in all contexts.

### Tool 6: Image to ASCII Art

#### Option A: ascii-image-converter (Recommended, Modern)
```bash
# Install
sudo snap install ascii-image-converter
# OR: go install github.com/TheZoraiz/ascii-image-converter@latest

# Usage
ascii-image-converter image.png                  # Basic
ascii-image-converter image.png -C               # Color output
ascii-image-converter image.png -d 60,30         # Set dimensions
ascii-image-converter image.png -b               # Braille characters
ascii-image-converter image.png -n               # Negative/inverted
ascii-image-converter https://url/image.jpg      # Direct URL
ascii-image-converter image.png --save-txt out   # Save as text
```

#### Option B: jp2a (Lightweight, JPEG Only)
```bash
sudo apt install jp2a -y
jp2a --width=80 image.jpg
jp2a --colors image.jpg              # Colorized
```

### Tool 7: Search Pre-Made ASCII Art

#### Source A: ascii.co.uk (Recommended)
Large collection organized by subject. Art in HTML `<pre>` tags.

**Fetch & extract:**
```bash
# 1. Fetch page
curl -s 'https://ascii.co.uk/art/cat' -o /tmp/ascii_art.html

# 2. Extract art from <pre> tags
python3 << 'EOF'
import re, html
with open('/tmp/ascii_art.html') as f:
    text = f.read()
arts = re.findall(r'<pre[^>]*>(.*?)</pre>', text, re.DOTALL)
for art in arts:
    clean = re.sub(r'<[^>]+>', '', art)
    clean = html.unescape(clean).strip()
    if len(clean) > 30:
        print(clean)
        print('\n---\n')
EOF
```

**Available subjects:**
- Animals: `cat`, `dog`, `horse`, `bird`, `fish`, `dragon`, `snake`, `rabbit`, `elephant`, `dolphin`, `butterfly`, `owl`, `wolf`, `bear`, `penguin`, `turtle`
- Objects: `car`, `ship`, `airplane`, `rocket`, `guitar`, `computer`, `coffee`, `beer`, `cake`, `house`, `castle`, `sword`, `crown`, `key`
- Nature: `tree`, `flower`, `sun`, `moon`, `star`, `mountain`, `ocean`, `rainbow`
- Characters: `skull`, `robot`, `angel`, `wizard`, `pirate`, `ninja`, `alien`
- Holidays: `christmas`, `halloween`, `valentine`

**Tips:** Preserve artist signatures; multiple pieces per page — pick best.

#### Source B: GitHub Octocat API (Fun)
```bash
curl -s https://api.github.com/octocat
```
Random Octocat with wise quote. No auth needed.

### Tool 8: Fun ASCII Utilities (via curl)

#### QR Codes as ASCII Art
```bash
curl -s "qrenco.de/Hello+World"
curl -s "qrenco.de/https://example.com"
```

#### Weather as ASCII Art
```bash
curl -s "wttr.in/London"          # Full weather with ASCII graphics
curl -s "wttr.in/Moon"            # Moon phase in ASCII art
curl -s "v2.wttr.in/London"       # Detailed version
```

### Tool 9: LLM-Generated Custom Art (Fallback)

When tools above don't suffice, generate with Unicode characters:

#### Character Palette
**Box Drawing:** `╔ ╗ ╚ ╝ ║ ═ ╠ ╣ ╦ ╩ ╬ ┌ ┐ └ ┘ │ ─ ├ ┤ ┬ ┴ ┼ ╭ ╮ ╰ ╯`

**Block Elements:** `░ ▒ ▓ █ ▄ ▀ ▌ ▐ ▖ ▗ ▘ ▝ ▚ ▞`

**Geometric & Symbols:** `◆ ◇ ◈ ● ○ ◉ ■ □ ▲ △ ▼ ▽ ★ ☆ ✦ ✧ ◀ ▶ ◁ ▷ ⬡ ⬢ ⌂`

**Rules:** Max width 60 chars, max height 15 lines (banners) / 25 (scenes). Monospace only.

### Decision Flow
1. **Text as banner** → pyfiglet (installed) or asciified API (curl)
2. **Wrap message in character** → cowsay
3. **Decorative border/frame** → boxes (combine with pyfiglet/asciified)
4. **Art of specific thing** (cat, rocket) → ascii.co.uk via curl + parsing
5. **Convert image to ASCII** → ascii-image-converter or jp2a
6. **QR code** → qrenco.de via curl
7. **Weather/moon art** → wttr.in via curl
8. **Custom/creative** → LLM generation with Unicode palette
9. **Tool not installed** → install it, or fall back to next option

---

## ASCII Video Production Pipeline

> **Source:** Absorbed from `ascii-video` skill. Convert video/audio to colored ASCII MP4/GIF.

### When to Use
- ASCII video, text art video, terminal-style video
- Character art animation, retro text visualization
- Audio visualizer in ASCII, matrix-style effects
- Converting video to ASCII art

### Creative Standard
**This is visual art.** ASCII characters are the medium; cinema is the standard.
- Before coding: articulate creative concept (mood, visual story, color world, character texture)
- First-render excellence: visually striking without revision rounds
- Go beyond reference vocabulary: combine, modify, invent new patterns
- Cohesive aesthetic: all scenes connected by unifying visual language
- Dense, layered, considered: multi-grid composition, per-scene variation, intentional color

### Modes
| Mode | Input | Output | Reference |
|------|-------|--------|-----------|
| **Video-to-ASCII** | Video file | ASCII recreation | `references/inputs.md` § Video Sampling |
| **Audio-reactive** | Audio file | Generative visuals driven by audio | `references/inputs.md` § Audio Analysis |
| **Generative** | None (or seed) | Procedural ASCII animation | `references/effects.md` |
| **Hybrid** | Video + audio | ASCII video with audio-reactive overlays | Both input refs |
| **Lyrics/text** | Audio + text/SRT | Timed text with visual effects | `references/inputs.md` § Text/Lyrics |
| **TTS narration** | Text quotes + TTS API | Narrated testimonial video | `references/inputs.md` § TTS Integration |

### Stack
Single self-contained Python script. No GPU required.
| Layer | Tool | Purpose |
|-------|------|---------|
| Core | Python 3.10+, NumPy | Math, array ops, vectorized effects |
| Signal | SciPy | FFT, peak detection (audio modes) |
| Imaging | Pillow (PIL) | Font rasterization, frame decoding, image I/O |
| Video I/O | ffmpeg (CLI) | Decode input, encode output, mux audio |
| Parallel | concurrent.futures | N workers for batch/clip rendering |
| TTS | ElevenLabs API (optional) | Generate narration clips |
| Optional | OpenCV | Video frame sampling, edge detection |

### Pipeline Architecture
```
INPUT → ANALYZE → SCENE_FN → TONEMAP → SHADE → ENCODE
```
1. **INPUT** — Load/decode source (video frames, audio samples, images, or nothing)
2. **ANALYZE** — Extract per-frame features (audio bands, video luminance/edges, motion)
3. **SCENE_FN** — Render to pixel canvas (`uint8 H,W,3`). Composes multiple char grids via `_render_vf()` + pixel blend modes
4. **TONEMAP** — Percentile-based adaptive brightness normalization
5. **SHADE** — Post-processing via `ShaderChain` + `FeedbackBuffer`
6. **ENCODE** — Pipe raw RGB frames to ffmpeg for H.264/GIF encoding

### Creative Direction

#### Aesthetic Dimensions
| Dimension | Options | Reference |
|-----------|---------|-----------|
| **Character palette** | Density ramps, block elements, symbols, scripts (katakana, Greek, runes, braille), project-specific | `architecture.md` § Palettes |
| **Color strategy** | HSV, OKLAB/OKLCH, discrete RGB palettes, auto-generated harmony, monochrome, temperature | `architecture.md` § Color System |
| **Background texture** | Sine fields, fBM noise, domain warp, voronoi, reaction-diffusion, cellular automata, video | `effects.md` |
| **Primary effects** | Rings, spirals, tunnel, vortex, waves, interference, aurora, fire, SDFs, strange attractors | `effects.md` |
| **Particles** | Sparks, snow, rain, bubbles, runes, orbits, flocking boids, flow-field followers, trails | `effects.md` § Particles |
| **Shader mood** | Retro CRT, clean modern, glitch art, cinematic, dreamy, industrial, psychedelic | `shaders.md` |
| **Grid density** | xs(8px) through xxl(40px), mixed per layer | `architecture.md` § Grid System |
| **Coordinate space** | Cartesian, polar, tiled, rotated, fisheye, Möbius, domain-warped | `effects.md` § Transforms |
| **Feedback** | Zoom tunnel, rainbow trails, ghostly echo, rotating mandala, color evolution | `composition.md` § Feedback |
| **Masking** | Circle, ring, gradient, text stencil, animated iris/wipe/dissolve | `composition.md` § Masking |
| **Transitions** | Crossfade, wipe, dissolve, glitch cut, iris, mask-based reveal | `shaders.md` § Transitions |

#### Per-Section Variation (MANDATORY)
Never use same config for entire video. For each section/scene:
- Different background effect (or compose 2-3)
- Different character palette (match mood)
- Different color strategy (or at minimum different hue)
- Vary shader intensity
- Different particle types if particles active

#### Project-Specific Invention
For every project, invent at least one:
- Custom character palette matching theme
- Custom background effect (combine/modify building blocks)
- Custom color palette (discrete RGB set matching brand/mood)
- Custom particle character set
- Novel scene transition or visual moment

### Workflow

#### Step 1: Creative Vision
Articulate before any code:
- Mood/atmosphere: energetic, meditative, chaotic, elegant, ominous?
- Visual story: build tension? transform? dissolve?
- Color world: warm/cool, monochrome, neon, earth tones, dominant hue?
- Character texture: dense data, sparse stars, organic dots, geometric blocks?
- What makes THIS different?
- Emotional arc: open with energy, build to climax, resolve?

#### Step 2: Technical Design
- **Mode** — which of 6 modes
- **Resolution** — landscape 1920x1080 (default), portrait 1080x1920, square 1080x1080 @ 24fps
- **Hardware detection** — auto-detect cores/RAM, set quality profile (`references/optimization.md`)
- **Sections** — map timestamps to scene functions with effect/palette/color/shader config
- **Output format** — MP4 (default), GIF (640x360 @ 15fps), PNG sequence

#### Step 3: Build the Script
Single Python file. Components (with references):
1. Hardware detection + quality profile → `references/optimization.md`
2. Input loader → `references/inputs.md`
3. Feature analyzer — audio FFT, video luminance, or synthetic
4. Grid + renderer — multi-density grids with bitmap cache → `references/architecture.md`
5. Character palettes — multiple per project → `references/architecture.md` § Palettes
6. Color system — HSV + discrete RGB + harmony → `references/architecture.md` § Color
7. Scene functions — each returns `canvas (uint8 H,W,3)` → `references/scenes.md`
8. Tonemap — adaptive brightness → `references/composition.md`
9. Shader pipeline — `ShaderChain` + `FeedbackBuffer` → `references/shaders.md`
10. Scene table + dispatcher — time → scene function + config → `references/scenes.md`
11. Parallel encoder — N-worker clip rendering with ffmpeg pipes
12. Main — orchestrate full pipeline

#### Step 4: Quality Verification
- **Test frames first** — render single frames at key timestamps
- **Brightness check** — `canvas.mean() > 8` for all ASCII content
- **Visual coherence** — do all scenes feel like same video?
- **Creative vision check** — matches Step 1 concept? If generic, go back

### Critical Implementation Notes

#### Brightness — Use `tonemap()`, Not Linear Multipliers
ASCII on black is inherently dark. **Never use `canvas * N`** — clips highlights. Use adaptive tonemap:
```python
def tonemap(canvas, gamma=0.75):
    f = canvas.astype(np.float32)
    lo, hi = np.percentile(f[::4, ::4], [1, 99.5])
    if hi - lo < 10: hi = lo + 10
    f = np.clip((f - lo) / (hi - lo), 0, 1) ** gamma
    return (f * 255).astype(np.uint8)
```
Pipeline: `scene_fn() → tonemap() → FeedbackBuffer → ShaderChain → ffmpeg`
Per-scene gamma: default 0.75, solarize 0.55, posterize 0.50, bright scenes 0.85. Use `screen` blend (not `overlay`) for dark layers.

#### Font Cell Height
macOS Pillow: `textbbox()` returns wrong height. Use `font.getmetrics()`: `cell_height = ascent + descent`. See `references/troubleshooting.md`.

#### ffmpeg Pipe Deadlock
Never `stderr=subprocess.PIPE` with long-running ffmpeg — buffer fills at 64KB and deadlocks. Redirect to file.

#### Font Compatibility
Not all Unicode chars render in all fonts. Validate palettes at init — render each char, check for blank output.

#### Per-Clip Architecture
For segmented videos (quotes, scenes, chapters), render each as separate clip file for parallel rendering and selective re-rendering.

### Performance Targets
| Component | Budget |
|-----------|--------|
| Feature extraction | 1-5ms |
| Effect function | 2-15ms |
| Character render | 80-150ms (bottleneck) |
| Shader pipeline | 5-25ms |
| **Total** | ~100-200ms/frame |

### References (from original skill)
- `references/architecture.md` — Grid system, resolution presets, font selection, 20+ char palettes, color system, `_render_vf()`, GridLayer
- `references/composition.md` — 20 pixel blend modes, `blend_canvas()`, multi-grid composition, adaptive `tonemap()`, `FeedbackBuffer`, `PixelBlendStack`, masking/stencil
- `references/effects.md` — Effect building blocks: value fields, hue fields, noise/fBM/domain warp, voronoi, reaction-diffusion, cellular automata, SDFs, strange attractors, particle systems, coordinate transforms, temporal coherence
- `references/shaders.md` — `ShaderChain`, 38 shader catalog, audio-reactive scaling, transitions, tint presets, output encoding, terminal rendering
- `references/scenes.md` — Scene protocol, `Renderer` class, `SCENES` table, `render_clip()`, beat-synced cutting, parallel rendering, design patterns, complete scene examples
- `references/inputs.md` — Audio analysis (FFT, bands, beats), video sampling, image conversion, text/lyrics, TTS integration
- `references/optimization.md` — Hardware detection, quality profiles, vectorized patterns, parallel rendering, memory management, performance budgets
- `references/troubleshooting.md` — NumPy broadcasting traps, blend mode pitfalls, multiprocessing/pickling, brightness diagnostics, ffmpeg issues, font problems

---

## Related Skills

- **`creative-diagrams-visualizations`** — Structured diagrams (architecture, infographics, comics)
- **`creative-generative-ai`** — ComfyUI/Stable Diffusion for AI image generation
- **`creative-coding-animation`** — Manim (math animations), p5js (creative coding)
- **`creative-design-systems`** — Design system references for visual consistency
