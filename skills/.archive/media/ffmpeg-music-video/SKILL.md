---
name: ffmpeg-music-video
description: Build complete instrumental music videos from scratch using only ffmpeg on Android/Termux. Synthesize audio from sine waves, generate spectrograms/visuals, overlay title cards with drawtext, and compile everything into an MP4 slideshow — no external samples, no GPU, no DAW needed.
version: 1.0.0
author: Hermes Agent
metadata:
  hermes:
    tags: [ffmpeg, music, video, instrumental, synthesis, termux, android, spectrogram, aevalsrc, slideshow]
---

# ffmpeg Music Video — Full Pipeline (Termux/Android)

Generate a complete instrumental music video using **only ffmpeg** — no external audio samples, no DAW, no GPU required. Built and tested on Android/Termux with Kimi K2.6 via Hermes Agent.

## When to Use

- You want instrumental music + visuals + title cards compiled into an MP4
- You're on Termux/Android with no DAW, no Go, no Python audio libs
- You need a hackathon-ready multimedia deliverable from a single terminal
- You want to avoid licensing issues by synthesizing everything from scratch

## Prerequisites

- `ffmpeg` with libx264 and libmp3lame support (`pkg install ffmpeg` on Termux)
- A TrueType font for drawtext (e.g., `/system/fonts/CarroisGothicSC-Regular.ttf` on Android)
- ~50MB free disk space for a 45s 1080p output

## Pipeline Overview

```
Step 1: Synthesize instrumental audio (aevalsrc sine waves)
Step 2: Encode to MP3
Step 3: Generate spectrogram/waveform visuals from audio
Step 4: Overlay title cards with drawtext
Step 5: Compile slideshow + audio into final MP4
```

---

## Step 1: Synthesize Instrumental Audio

Use `aevalsrc` to generate sine-wave layers. Build bass, arpeggio, lead, and texture tracks separately, then mix.

```bash
ffmpeg -f lavfi -i 'aevalsrc=0.4*sin(2*PI*65*t)+0.25*sin(2*PI*130*t)+0.15*sin(2*PI*196*t):s=48000:d=45' \
  -f lavfi -i 'aevalsrc=0.12*sin(2*PI*440*t)*sin(2*PI*2*t+PI/2)*0.5+0.5:s=48000:d=45' \
  -f lavfi -i 'aevalsrc=0.08*sin(2*PI*880*t)*sin(2*PI*8*t+PI/2)*0.5+0.5:s=48000:d=45' \
  -f lavfi -i 'aevalsrc=0.06*sin(2*PI*3000*t)*sin(2*PI*4*t+PI/2)*0.5+0.5:s=48000:d=45' \
  -filter_complex '
    [0:a]lowpass=f=250[bass];
    [1:a]bandpass=f=600:w=500[arp];
    [2:a]highpass=f=700[lead];
    [3:a]highpass=f=2000[noise];
    [bass][arp][lead][noise]amix=inputs=4:duration=longest,volume=4.0,dynaudnorm,afade=t=out:st=40:d=5
  ' -t 45 ~/beat.wav
```

**Layer breakdown:**
| Layer | Frequency | Role | Filter |
|-------|-----------|------|--------|
| 0 | 65Hz + 130Hz + 196Hz | Bass drone | `lowpass=f=250` |
| 1 | 440Hz gated at 2Hz | Arpeggio pulse | `bandpass=f=600:w=500` |
| 2 | 880Hz gated at 8Hz | Lead synth | `highpass=f=700` |
| 3 | 3kHz gated at 4Hz | Texture/noise | `highpass=f=2000` |

**Key filters:**
- `amix=inputs=4:duration=longest,volume=4.0` — mixes all layers
- `dynaudorm` — auto-normalizes volume
- `afade=t=out:st=40:d=5` — 5-second fadeout starting at 40s

**Pitfall — `max()` does not work in aevalsrc expressions** on some ffmpeg builds. Use `sin(2*PI*X*t+PI/2)*0.5+0.5` as a gated amplitude envelope instead.

---

## Step 2: Encode to MP3

```bash
ffmpeg -i ~/beat.wav -c:a libmp3lame -b:a 192k ~/beat.mp3
```

---

## Step 3: Generate Visuals from Audio

### Fire Spectrogram (good for dark backgrounds)
```bash
ffmpeg -i ~/beat.mp3 -lavfi "showspectrumpic=s=1920x1080:mode=separate:color=fire" \
  -update 1 -frames:v 1 ~/spectrogram_fire.png
```

### Neon Channel Spectrogram
```bash
ffmpeg -i ~/beat.mp3 -lavfi "showspectrumpic=s=1920x1080:mode=combined:color=channel" \
  -update 1 -frames:v 1 ~/spectrogram_neon.png
```

### Simple Waveform
```bash
ffmpeg -i ~/beat.mp3 -lavfi "showwavespic=s=1920x540:colors=cyan|magenta" \
  -update 1 -frames:v 1 ~/waveform.png
```

**Critical:** Always use `-update 1` when writing single PNG frames with image2 muxer, or ffmpeg errors with "Use a pattern such as %03d..."

---

## Step 4: Overlay Title Cards with drawtext

Use Android system fonts or install a TTF. On Termux, Android fonts live at `/system/fonts/`.

```bash
FONT="/system/fonts/CarroisGothicSC-Regular.ttf"

ffmpeg -i ~/spectrogram_fire.png -vf \
  "drawtext=fontfile=$FONT:text='TITLE LINE 1':fontsize=120:fontcolor=white:x=(w-text_w)/2:y=60:box=1:boxcolor=black@0.7:boxborderw=20,\
   drawtext=fontfile=$FONT:text='Subtitle':fontsize=50:fontcolor=cyan:x=(w-text_w)/2:y=200:box=1:boxcolor=black@0.5:boxborderw=12" \
  -update 1 -frames:v 1 ~/title_card.png
```

**Positioning macros:**
- `x=(w-text_w)/2` — horizontally centered
- `y=h-text_h-60` — 60px from bottom
- `box=1:boxcolor=black@0.7` — semi-transparent background behind text

---

## Step 5: Compile Slideshow + Audio into MP4

Create a concat script file listing your title cards and durations:

```bash
cat > ~/slideshow.txt << 'EOF'
file '/data/data/com.termux/files/home/title1.png'
duration 12
file '/data/data/com.termux/files/home/title2.png'
duration 12
file '/data/data/com.termux/files/home/title3.png'
duration 12
file '/data/data/com.termux/files/home/title1.png'
duration 9
EOF
```

**Critical:** Use absolute paths in the concat file. Relative paths fail inside the concat demuxer on some ffmpeg builds.

Compile:
```bash
ffmpeg -f concat -safe 0 -i ~/slideshow.txt -i ~/beat.mp3 \
  -vf "format=yuv420p,scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" \
  -shortest -c:v libx264 -preset fast -crf 23 -c:a aac -b:a 192k ~/music_video.mp4
```

**Video filter explanation:**
- `format=yuv420p` — required for H.264 compatibility
- `scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2` — scales image down to fit 1920x1080, then pads with black bars to exactly 1080p. Prevents aspect ratio distortion.
- `-shortest` — trims video to match audio length (or vice versa)
- `-preset fast` — balances encode speed vs file size on mobile CPU

---

## Complete One-Shot Script

```bash
#!/data/data/com.termux/files/usr/bin/env bash
set -e

DUR=45
FONT="/system/fonts/CarroisGothicSC-Regular.ttf"

# 1. Synthesize audio
ffmpeg -f lavfi -i 'aevalsrc=0.4*sin(2*PI*65*t)+0.25*sin(2*PI*130*t)+0.15*sin(2*PI*196*t):s=48000:d='$DUR \
  -f lavfi -i 'aevalsrc=0.12*sin(2*PI*440*t)*sin(2*PI*2*t+PI/2)*0.5+0.5:s=48000:d='$DUR \
  -f lavfi -i 'aevalsrc=0.08*sin(2*PI*880*t)*sin(2*PI*8*t+PI/2)*0.5+0.5:s=48000:d='$DUR \
  -f lavfi -i 'aevalsrc=0.06*sin(2*PI*3000*t)*sin(2*PI*4*t+PI/2)*0.5+0.5:s=48000:d='$DUR \
  -filter_complex '[0:a]lowpass=f=250[b];[1:a]bandpass=f=600:w=500[a];[2:a]highpass=f=700[l];[3:a]highpass=f=2000[n];[b][a][l][n]amix=inputs=4:duration=longest,volume=4.0,dynaudnorm,afade=t=out:st=40:d=5' \
  -t $DUR beat.wav

ffmpeg -i beat.wav -c:a libmp3lame -b:a 192k beat.mp3

# 2. Visuals
ffmpeg -i beat.mp3 -lavfi "showspectrumpic=s=1920x1080:mode=separate:color=fire" -update 1 -frames:v 1 vis1.png
ffmpeg -i beat.mp3 -lavfi "showspectrumpic=s=1920x1080:mode=combined:color=channel" -update 1 -frames:v 1 vis2.png

# 3. Title cards
ffmpeg -i vis1.png -vf "drawtext=fontfile=$FONT:text='AGENT MODE':fontsize=120:fontcolor=white:x=(w-text_w)/2:y=60:box=1:boxcolor=black@0.7:boxborderw=20,drawtext=fontfile=$FONT:text='Hermes Agent x Kimi K2.6':fontsize=50:fontcolor=cyan:x=(w-text_w)/2:y=200:box=1:boxcolor=black@0.5:boxborderw=12" -update 1 -frames:v 1 title1.png
ffmpeg -i vis2.png -vf "drawtext=fontfile=$FONT:text='NO LYRICS':fontsize=120:fontcolor=cyan:x=(w-text_w)/2:y=80:box=1:boxcolor=black@0.7:boxborderw=20,drawtext=fontfile=$FONT:text='Pure Signal':fontsize=60:fontcolor=magenta:x=(w-text_w)/2:y=220:box=1:boxcolor=black@0.5:boxborderw=12" -update 1 -frames:v 1 title2.png

# 4. Slideshow list
cat > slideshow.txt <<EOF
file '$(pwd)/title1.png'
duration 15
file '$(pwd)/title2.png'
duration 15
file '$(pwd)/title1.png'
duration 15
EOF

# 5. Compile video
ffmpeg -f concat -safe 0 -i slideshow.txt -i beat.mp3 \
  -vf "format=yuv420p,scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" \
  -shortest -c:v libx264 -preset fast -crf 23 -c:a aac -b:a 192k music_video.mp4

echo "Done: music_video.mp4"
```

---

## Pitfalls & Lessons Learned

1. **`max()` fails in aevalsrc** — Use `sin(phase+PI/2)*0.5+0.5` as an amplitude gate instead of `max(0, sin(...))`.
2. **`-update 1` is required** for single-frame PNG output via image2 muxer. Without it, ffmpeg demands `%03d` pattern filenames.
3. **Absolute paths in concat files** — Relative paths often fail in the concat demuxer on Android/Termux.
4. **`command -v` vs `which`** — On some Termux setups, `/bin/which` is missing but `command -v` works for path detection.
5. **`pkg install golang`** then `go install` is the path to Songsee if you want true spectrograms; but ffmpeg's `showspectrumpic` works without any Go dependency for basic visuals.
6. **`scale=1920:1080:force_original_aspect_ratio=decrease,pad=...`** is the safest way to force exactly 1080p without distorting your source images.
7. **`dynaudnorm`** before `afade` prevents the fade from clipping or dropping too early.
8. **Font selection matters** — Android system fonts vary by device. If drawtext fails, list available fonts with `ls /system/fonts/*.ttf`.

---

## Variations

- **Longer track:** Increase `d=45` and `t 45` values; adjust fade timing
- **Different mood:** Swap frequencies (e.g., 110Hz + 220Hz for darker, 880Hz + 1760Hz for brighter)
- **More layers:** Add more `-f lavfi -i` inputs and increase `amix=inputs=N`
- **Tempo shift:** Change the gate frequencies (e.g., `sin(2*PI*4*t)` = 4Hz gate = 240 BPM feel)
- **Reverb:** Add `aecho=0.8:0.9:60:0.3` after each layer before mixing
- **Color palette:** In `showspectrumpic`, try `color=intensity`, `color=fire`, `color=channel`

---

## Related Skills

- `songsee` — For professional 9-panel audio analysis grids (requires Go)
- `songwriting-and-ai-music` — For lyric writing and Suno/HeartMuLa prompts
- `heartmula` — For full AI music generation with vocals (requires GPU)
