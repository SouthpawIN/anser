---
name: ai-music-generation
description: "AI music generation and audio analysis: open-source generation (HeartMuLa), commercial prompt engineering (Suno), and audio visualization (songsee)."
version: 1.0.0
author: Hermes Agent
license: MIT
tags: ["music", "audio", "generation", "suno", "heartmula", "visualization", "spectrogram", "creative"]
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [music, audio, generation, suno, heartmula, visualization, spectrogram, creative]
    category: creative
    related_skills: []
---

# AI Music Generation & Audio Analysis

Unified skill for AI-powered music creation and audio analysis. Covers three complementary approaches:

1. **Open-Source Local Generation** (`heartmula`): HeartMuLa family models for local/offline song generation from lyrics + tags (Apache-2.0, comparable to Suno)
2. **Commercial Prompt Engineering** (`songwriting-and-ai-music`): Songwriting craft + Suno AI prompt engineering, metatags, phonetic tricks, and production pipelines
3. **Audio Analysis & Visualization** (`songsee`): Spectrograms, MFCC, chroma, tempogram, and other audio feature visualizations via CLI

Load this skill when the user wants to:
- Generate music locally with open-source models (HeartMuLa)
- Create professional Suno prompts with proper structure, metatags, and phonetic optimization
- Visualize audio files as spectrograms, waveforms, or feature grids
- Build complete promo packages: lyrics → audio → cover art → spectrograms

---

## Quick Decision Guide

| Goal | Use This Section |
|------|------------------|
| Generate songs locally, offline, Apache-2.0 | [Open-Source Generation (HeartMuLa)](#open-source-generation-heartmula) |
| Get best results from Suno AI (commercial) | [Commercial Prompt Engineering (Suno)](#commercial-prompt-engineering-suno) |
| Analyze/visualize existing audio files | [Audio Visualization (songsee)](#audio-visualization-songsee) |
| Full pipeline: lyrics → audio → visuals | [Promo Package Pipeline](#promo-package-pipeline) |

---

## Open-Source Generation (HeartMuLa)

> **Source:** Absorbed from `heartmula` skill. HeartMuLa is a family of open-source music foundation models (Apache-2.0) generating music from lyrics + tags with multilingual support.

### When to Use
- User wants open-source Suno alternative
- User needs local/offline music generation
- User has GPU with ≥8GB VRAM (16GB+ recommended)
- User asks about HeartMuLa, heartlib, or HeartCodec

### Hardware Requirements
| Tier | VRAM | Notes |
|------|------|-------|
| Minimum | 8GB | With `--lazy_load true` (sequential load/unload) |
| Recommended | 16GB+ | Comfortable single-GPU usage |
| Multi-GPU | Split | `--mula_device cuda:0 --codec_device cuda:1` |
| 3B model + lazy_load | ~6.2GB peak | Most accessible config |

**No NVIDIA GPU?** Use CPU (`--mula_device cpu --codec_device cpu`) — extremely slow (30-60+ min/song). Prefer cloud GPU (Colab T4, Lambda Labs) or online demo at https://heartmula.github.io/.

### Installation
```bash
cd ~/
git clone https://github.com/HeartMuLa/heartlib.git
cd heartlib

# Python 3.10 REQUIRED
uv venv --python 3.10 .venv
. .venv/bin/activate
uv pip install -e .

# Fix dependency conflicts (as of Feb 2026)
uv pip install --upgrade datasets transformers
```

### Required Source Patches (transformers 5.x compatibility)
**Patch 1** — `src/heartlib/heartmula/modeling_heartmula.py`, in `setup_caches()` after `reset_caches` try/except:
```python
from torchtune.models.llama3_1._position_embeddings import Llama3ScaledRoPE
for module in self.modules():
    if isinstance(module, Llama3ScaledRoPE) and not module.is_cache_built:
        module.rope_init()
        module.to(device)
```

**Patch 2** — `src/heartlib/pipelines/music_generation.py`, add `ignore_mismatched_sizes=True` to ALL `HeartCodec.from_pretrained()` calls (2 locations).

### Model Downloads
```bash
hf download --local-dir './ckpt' 'HeartMuLa/HeartMuLaGen'
hf download --local-dir './ckpt/HeartMuLa-oss-3B' 'HeartMuLa/HeartMuLa-oss-3B-happy-new-year'
hf download --local-dir './ckpt/HeartCodec-oss' 'HeartMuLa/HeartCodec-oss-20260123'
```

### Generation Command
```bash
cd heartlib
. .venv/bin/activate
python ./examples/run_music_generation.py \
  --model_path=./ckpt \
  --version="3B" \
  --lyrics="./assets/lyrics.txt" \
  --tags="./assets/tags.txt" \
  --save_path="./assets/output.mp3" \
  --lazy_load true
```

### Input Formats
**Tags** (comma-separated, no spaces):
```
piano,happy,wedding,synthesizer,romantic
rock,energetic,guitar,drums,male-vocal
```

**Lyrics** (bracketed structural tags):
```
[Intro]

[Verse]
Your lyrics here...

[Chorus]
Chorus lyrics...

[Bridge]
Bridge lyrics...

[Outro]
```

### Key Parameters
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--max_audio_length_ms` | 240000 | Max length (240s = 4 min) |
| `--topk` | 50 | Top-k sampling |
| `--temperature` | 1.0 | Sampling temperature |
| `--cfg_scale` | 1.5 | Classifier-free guidance |
| `--lazy_load` | false | Sequential model load/unload (saves VRAM) |
| `--mula_dtype` | bfloat16 | HeartMuLa dtype (bf16 recommended) |
| `--codec_dtype` | float32 | HeartCodec dtype (fp32 for quality!) |

### Performance & Output
- **RTF ≈ 1.0** — 4-min song takes ~4 minutes on GPU
- **Output**: MP3, 48kHz stereo, 128kbps

### Critical Pitfalls
1. **NEVER use bf16 for HeartCodec** — degrades audio quality. Use fp32.
2. **Tags may be ignored** — known issue (#90). Lyrics dominate; experiment with tag ordering.
3. **Triton not on macOS** — Linux/CUDA only for GPU acceleration.
4. **RTX 5080 incompatibility** reported upstream.

### Links
- Repo: https://github.com/HeartMuLa/heartlib
- Models: https://huggingface.co/HeartMuLa
- Paper: https://arxiv.org/abs/2601.10547

---

## Commercial Prompt Engineering (Suno)

> **Source:** Absorbed from `songwriting-and-ai-music` skill. Professional songwriting craft + Suno-specific prompt engineering, metatags, phonetic tricks, and production workflows.

### When to Use
- User wants to write songs for Suno AI generation
- User needs help with song structure, rhyme, meter, emotional arc
- User wants to create parody/adaptation lyrics
- User asks for Suno prompt engineering, metatags, or phonetic tricks
- User wants a complete promo package: lyrics → audio → visuals

### Song Structure Templates
| Structure | Pattern | Best For |
|-----------|---------|----------|
| Pop/Rock | ABABCB | Verse/Chorus/Verse/Chorus/Bridge/Chorus |
| Jazz/Ballad | AABA | Verse/Verse/Bridge/Verse (refrain) |
| Simple | ABAB | Alternating verse/chorus |
| Folk/Story | AAA | Strophic, no chorus |

**Six Building Blocks:** Intro → Verse → Pre-Chorus → Chorus → Bridge → Outro. Use what serves the emotion.

### Suno Style/Genre Description Formula
```
Genre + Mood + Era + Instruments + Vocal Style + Production + Dynamics
```

**BAD:** `"sad rock song"`
**GOOD:** `"Cinematic orchestral spy thriller, 1960s Cold War era, smoky sultry female vocalist, big band jazz, brass section with trumpets and french horns, sweeping strings, minor key, vintage analog warmth"`

**DESCRIBE THE JOURNEY:**
```
"Begins as a haunting whisper over sparse piano. Gradually layers in muted brass. Builds through the chorus with full orchestra. Second verse erupts with raw belting intensity. Outro strips back to a lone piano and a fragile whisper fading to silence."
```

### Metatags (place in `[brackets]` inside lyrics field)

| Category | Tags |
|----------|------|
| **Structure** | `[Intro] [Verse] [Pre-Chorus] [Chorus] [Post-Chorus] [Hook] [Bridge] [Interlude] [Instrumental] [Guitar Solo] [Breakdown] [Build-up] [Outro] [Silence] [End]` |
| **Vocal Performance** | `[Whispered] [Spoken Word] [Belted] [Falsetto] [Powerful] [Soulful] [Raspy] [Breathy] [Smooth] [Gritty] [Staccato] [Legato] [Vibrato] [Melismatic] [Harmonies] [Choir] [Harmonized Chorus]` |
| **Dynamics** | `[High Energy] [Low Energy] [Building Energy] [Explosive] [Emotional Climax] [Gradual swell] [Orchestral swell] [Quiet arrangement] [Falling tension] [Slow Down]` |
| **Gender** | `[Female Vocals] [Male Vocals]` |
| **Atmosphere** | `[Melancholic] [Euphoric] [Nostalgic] [Aggressive] [Dreamy] [Intimate] [Dark Atmosphere]` |
| **SFX** | `[Vinyl Crackle] [Rain] [Applause] [Static] [Thunder]` |

**Rules:** Put tags in BOTH style field AND lyrics for reinforcement. 5-8 tags per section max. No contradictions (`[Calm]` + `[Aggressive]` in same section).

### Custom Mode
- Always use Custom Mode (separate Style + Lyrics fields)
- Lyrics limit: ~3,000 chars (~40-60 lines)
- Always add structural tags — without them Suno defaults to flat verse/chorus

### Phonetic Tricks for AI Singers
| Technique | Example |
|-----------|---------|
| Phonetic respelling | `"through"` → `"thru"`, `"Nous"` → `"Noose"` |
| Hyphenate syllables | `"Re-search"`, `"bio-engineering"` |
| ALL CAPS | = louder, more intense |
| Vowel extension | `"lo-o-o-ove"` = sustained/melisma |
| Ellipses | `"I... need... you"` = dramatic pauses |
| Hyphenated stretch | `"ne-e-ed"` = emotional stretch |
| Numbers spelled out | `"24/7"` → `"twenty four seven"` |
| Acronyms spaced | `"AI"` → `"A I"` or `"A-I"` |

### Parody & Adaptation Workflow
1. Map original: syllables per line, rhyme scheme, stressed syllables, held notes
2. Match stressed syllables to same beats as original
3. Flex total syllable count by 1-2 unstressed syllables
4. On held notes, match VOWEL SOUND of original
5. Monosyllabic swaps in key spots: `Crime → Code`, `Snake → Noose`
6. Sing new words over original — if you stumble, revise

### Complete Workflow
1. Write concept/hook first — emotional core?
2. If adapting, map original structure (syllables, rhyme, stress)
3. Generate raw material — brainstorm freely
4. Draft lyrics into structure
5. Read/sing aloud — fix meter
6. Build Suno style description — paint the dynamic journey
7. Add metatags to lyrics for performance direction
8. Generate 3-5 variations minimum — treat as recording takes
9. Pick best, use Extend/Continue on promising sections
10. Keep happy accidents

**Expect ~3-5 generations per 1 good result. Revision is normal.**

---

## Audio Visualization (songsee)

> **Source:** Absorbed from `songsee` skill. Generate spectrograms and multi-panel audio feature visualizations from audio files via CLI.

### When to Use
- User wants to visualize audio files (spectrograms, waveforms, feature grids)
- User needs audio analysis for music production debugging
- User wants visual documentation of audio outputs
- User needs promo materials (spectrogram title cards for social media)

### Installation
```bash
go install github.com/steipete/songsee/cmd/songsee@latest
# Optional: ffmpeg for formats beyond WAV/MP3
```

### Quick Start
```bash
# Basic spectrogram
songsee track.mp3

# Save to file
songsee track.mp3 -o spectrogram.png

# Multi-panel visualization grid (all 9 types)
songsee track.mp3 --viz spectrogram,mel,chroma,hpss,selfsim,loudness,tempogram,mfcc,flux

# Time slice (start at 12.5s, 8s duration)
songsee track.mp3 --start 12.5 --duration 8 -o slice.jpg

# From stdin
cat track.mp3 | songsee - --format png -o out.png
```

### Visualization Types (`--viz`)
| Type | Description |
|------|-------------|
| `spectrogram` | Standard frequency spectrogram |
| `mel` | Mel-scaled spectrogram |
| `chroma` | Pitch class distribution |
| `hpss` | Harmonic/percussive separation |
| `selfsim` | Self-similarity matrix |
| `loudness` | Loudness over time |
| `tempogram` | Tempo estimation |
| `mfcc` | Mel-frequency cepstral coefficients |
| `flux` | Spectral flux (onset detection) |

Multiple types render as a grid in a single image.

### Common Flags
| Flag | Description |
|------|-------------|
| `--style` | Color palette: `classic`, `magma`, `inferno`, `viridis`, `gray` |
| `--width` / `--height` | Output dimensions |
| `--window` / `--hop` | FFT window and hop size |
| `--min-freq` / `--max-freq` | Frequency range filter |
| `--start` / `--duration` | Time slice of audio |
| `--format` | Output: `jpg` or `png` |
| `-o` | Output file path |

### Alternative: `ffmpeg` Native Filters (Zero Install)
If `songsee` not available, `ffmpeg` has built-in filters:

**Spectrogram:**
```bash
ffmpeg -i track.mp3 -lavfi "showspectrumpic=s=1920x1080:mode=separate:color=fire" -frames:v 1 spectrogram.png
```

**Waveform:**
```bash
ffmpeg -i track.mp3 -lavfi "showwavespic=s=1920x540:colors=cyan|magenta" -frames:v 1 waveform.png
```

**Title Card Overlay:**
```bash
ffmpeg -i spectrogram.png -vf "drawtext=fontfile=/path/to/font.ttf:text='SONG TITLE':fontsize=120:fontcolor=cyan:x=(w-text_w)/2:y=80:box=1:boxcolor=black@0.6" -frames:v 1 title_card.png
```

**Vertical Stack (Spectrogram + Waveform):**
```bash
ffmpeg -i top.png -i bottom.png -filter_complex vstack=inputs=2 combined.png
```

### Notes
- WAV/MP3 decoded natively; other formats need ffmpeg
- Output images can be inspected with `vision_analyze` for automated analysis
- Useful for comparing audio outputs, debugging synthesis, documenting pipelines

---

## Promo Package Pipeline (Lyrics → Audio → Visuals)

> **From `songwriting-and-ai-music` §9.** Complete creative deliverable for hackathons, social media drops, artist promos.

**Step 1 — Write lyrics with metatags** (see Suno section above)

**Step 2 — Generate spoken word / vocal reference**
```bash
Use text_to_speech tool with full lyrics as dramatic spoken delivery.
Gives immediate audio to share AND vocal timing reference.
```

**Step 3 — AI poster / cover art**
```bash
Use image_generate with cinematic prompt describing song's theme.
Include: mood, color palette, typography hints, symbolic imagery.
```

**Step 4 — Audio visualization (spectrogram / waveform)**
```bash
# Fire spectrogram
ffmpeg -i audio.mp3 -lavfi "showspectrumpic=s=1920x1080:mode=separate:color=fire" -frames:v 1 spectrogram.png

# Title card overlay
ffmpeg -i spectrogram.png -vf "drawtext=fontfile=/path/to/font.ttf:text='SONG TITLE':fontsize=120:fontcolor=cyan:x=(w-text_w)/2:y=80:box=1:boxcolor=black@0.6" -frames:v 1 title.png
```
See songsee section for full ffmpeg options.

**Step 5 — Compile**
- Lyric sheet (markdown)
- Audio file (TTS or AI-generated music)
- Cover art (image_generate)
- Spectrogram title cards (ffmpeg)

**Entire pipeline executes from terminal with zero DAW software.** Perfect for hackathons, rapid prototyping, AI-native music releases.

---

## Lessons Learned (Cross-Domain)

- **Dynamic arc > genre list** — "Whisper to roar to whisper" gives Suno a performance map; HeartMuLa tags work similarly
- **Keep some originals in parody** — adds recognizability and emotional weight
- **Bridge = transformation slot** — swap specific references for your theme's metaphors while keeping emotional function
- **Monosyllabic swaps** — cleanest way to maintain rhythm while changing meaning
- **Vocal persona description** — bigger difference than any single metatag
- **Strip AI patterns from lyrics** — see `humanizer` skill for de-slopping generated text
- **Craft serves art** — if a line breaks meter but hits harder, keep it

---

## Related Skills

- **`humanizer`** — Strip AI writing patterns from lyrics/promo copy before publishing
- **`image_generate`** — Built-in tool for cover art generation (used in promo pipeline)
- **`text_to_speech`** — Built-in tool for spoken word reference tracks
- **`youtube-content`** — Extract transcripts from music videos for analysis/remix
