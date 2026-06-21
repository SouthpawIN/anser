---
name: creative-diagrams-visualizations
description: "Creative diagrams & visualizations: architecture diagrams (SVG/HTML), infographics (21 layouts × 21 styles), knowledge comics (educational/biography/tutorial)."
version: 1.0.0
author: Hermes Agent
license: MIT
tags: ["creative", "diagrams", "visualizations", "architecture", "infographic", "comic", "svg", "html", "image-generation"]
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [creative, diagrams, visualizations, architecture, infographic, comic, svg, html, image-generation]
    category: creative
    related_skills: [creative-ascii, creative-generative-ai, creative-coding-animation, creative-design-systems]
---

# Creative Diagrams & Visualizations

Unified skill for creating structured visual content: technical architecture diagrams, information-rich infographics, and educational comics. All produce publication-ready outputs via different generation approaches.

**Contents:**
1. **Architecture Diagrams** — Dark-themed SVG diagrams as self-contained HTML (architecture-diagram)
2. **Infographics** — 21 layouts × 21 styles for data-rich visual summaries (baoyu-infographic)
3. **Knowledge Comics** — Educational/biographical/tutorial comics with character consistency (baoyu-comic)

Load this skill when the user wants to:
- Visualize software/system architecture, cloud infrastructure, microservice topology
- Create high-density visual summaries of complex topics
- Produce educational comics explaining concepts, biographies, or tutorials

---

## Quick Decision Guide

| Goal | Use This Section |
|------|------------------|
| Software/cloud architecture diagram | [Architecture Diagrams](#architecture-diagrams) |
| Data-rich infographic (comparison, timeline, hierarchy, etc.) | [Infographics](#infographics) |
| Educational comic (concept explanation, biography, tutorial) | [Knowledge Comics](#knowledge-comics) |
| ASCII/text-based diagrams | See `creative-ascii` skill |
| AI-generated images (ComfyUI, Stable Diffusion) | See `creative-generative-ai` skill |
| Design system references (Stripe, Linear, Vercel styles) | See `creative-design-systems` skill |

---

## Architecture Diagrams

> **Source:** Absorbed from `architecture-diagram` skill. Dark-themed SVG architecture/cloud/infra diagrams as HTML.

### When to Use
- Software system architecture (frontend/backend/database layers)
- Cloud infrastructure (VPC, regions, subnets, managed services)
- Microservice/service-mesh topology
- Database + API maps, deployment diagrams
- Any tech-infra subject fitting dark, grid-backed aesthetic

### Output
Single self-contained `.html` file with inline SVG + CSS. Works offline in any browser.

### Design System

#### Color Palette (Semantic Mapping)
| Component Type | Fill (rgba) | Stroke (Hex) |
|----------------|-------------|--------------|
| **Frontend** | `rgba(8, 51, 68, 0.4)` | `#22d3ee` (cyan-400) |
| **Backend** | `rgba(6, 78, 59, 0.4)` | `#34d399` (emerald-400) |
| **Database** | `rgba(76, 29, 149, 0.4)` | `#a78bfa` (violet-400) |
| **AWS/Cloud** | `rgba(120, 53, 15, 0.3)` | `#fbbf24` (amber-400) |
| **Security** | `rgba(136, 19, 55, 0.4)` | `#fb7185` (rose-400) |
| **Message Bus** | `rgba(251, 146, 60, 0.3)` | `#fb923c` (orange-400) |
| **External** | `rgba(30, 41, 59, 0.5)` | `#94a3b8` (slate-400) |

#### Background
- Slate-950 (`#020617`) with 40px grid pattern
- Font: JetBrains Mono (Google Fonts)

### Technical Implementation

#### Component Rendering
Rounded rectangles (`rx="6"`) with 1.5px strokes. **Double-rect masking** to prevent arrow bleed-through:
1. Draw opaque background rect (`#0f172a`)
2. Draw semi-transparent styled rect on top

#### Connection Rules
- **Z-Order:** Arrows drawn early (behind components)
- **Arrowheads:** SVG markers
- **Security flows:** Dashed lines, rose color (`#fb7185`)
- **Boundaries:**
  - Security groups: Dashed (`4,4`), rose
  - Regions: Large dashed (`8,4`), amber, `rx="12"`

#### Spacing & Layout
- Standard height: 60px (services), 80-120px (large)
- Vertical gap: ≥40px between components
- Message buses: In gaps between services, not overlapping
- **Legend:** CRITICAL — outside all boundaries, ≥20px below lowest boundary

### Document Structure (4 Parts)
1. **Header:** Title with pulsing dot + subtitle
2. **Main SVG:** Diagram in rounded border card
3. **Summary Cards:** 3-card grid below diagram
4. **Footer:** Minimal metadata

### Workflow
1. User describes system (components, connections, technologies)
2. Generate HTML following design system
3. Save as `.html` (e.g., `./my-architecture.html`)
4. User opens in browser: `open ./my-architecture.html` / `xdg-open`

### Template Reference
Full working template with all component types, arrows, boundaries, legend:
```bash
skill_view(name="creative-diagrams-visualizations", file_path="templates/architecture-template.html")
```

---

## Infographics

> **Source:** Absorbed from `baoyu-infographic` skill. 21 layouts × 21 styles (信息图, 可视化).

### When to Use
- Transform text/content into visual summaries
- Create "高密度信息大图" (high-density info graphics) or "信息图" (infographics)
- User provides content (text, file, URL, topic) + optionally layout/style/aspect/language

### Two Dimensions: Layout × Style
Freely combine any layout with any style.

#### Layout Gallery (21)
| Layout | Best For |
|--------|----------|
| `linear-progression` | Timelines, processes, tutorials |
| `binary-comparison` | A vs B, before/after, pros/cons |
| `comparison-matrix` | Multi-factor comparisons |
| `hierarchical-layers` | Pyramids, priority levels |
| `tree-branching` | Categories, taxonomies |
| `hub-spoke` | Central concept with related items |
| `structural-breakdown` | Exploded views, cross-sections |
| `bento-grid` | Multiple topics, overview (default) |
| `iceberg` | Surface vs hidden aspects |
| `bridge` | Problem-solution |
| `funnel` | Conversion, filtering |
| `isometric-map` | Spatial relationships |
| `dashboard` | Metrics, KPIs |
| `periodic-table` | Categorized collections |
| `comic-strip` | Narratives, sequences |
| `story-mountain` | Plot structure, tension arcs |
| `jigsaw` | Interconnected parts |
| `venn-diagram` | Overlapping concepts |
| `winding-roadmap` | Journey, milestones |
| `circular-flow` | Cycles, recurring processes |
| `dense-modules` | High-density modules, data-rich guides |

#### Style Gallery (21)
| Style | Description |
|-------|-------------|
| `craft-handmade` | Hand-drawn, paper craft (default) |
| `claymation` | 3D clay figures, stop-motion |
| `kawaii` | Japanese cute, pastels |
| `storybook-watercolor` | Soft painted, whimsical |
| `chalkboard` | Chalk on black board |
| `cyberpunk-neon` | Neon glow, futuristic |
| `bold-graphic` | Comic style, halftone |
| `aged-academia` | Vintage science, sepia |
| `corporate-memphis` | Flat vector, vibrant |
| `technical-schematic` | Blueprint, engineering |
| `origami` | Folded paper, geometric |
| `pixel-art` | Retro 8-bit |
| `ui-wireframe` | Grayscale interface mockup |
| `subway-map` | Transit diagram |
| `ikea-manual` | Minimal line art |
| `knolling` | Organized flat-lay |
| `lego-brick` | Toy brick construction |
| `pop-laboratory` | Blueprint grid, coordinate markers |
| `morandi-journal` | Hand-drawn doodle, warm Morandi tones |
| `retro-pop-grid` | 1970s retro pop art, Swiss grid |
| `hand-drawn-edu` | Macaron pastels, stick figures |

### Recommended Combinations
| Content Type | Layout + Style |
|--------------|----------------|
| Timeline/History | `linear-progression` + `craft-handmade` |
| Step-by-step | `linear-progression` + `ikea-manual` |
| A vs B | `binary-comparison` + `corporate-memphis` |
| Hierarchy | `hierarchical-layers` + `craft-handmade` |
| Overlap | `venn-diagram` + `craft-handmade` |
| Conversion | `funnel` + `corporate-memphis` |
| Cycles | `circular-flow` + `craft-handmade` |
| Technical | `structural-breakdown` + `technical-schematic` |
| Metrics/KPIs | `dashboard` + `corporate-memphis` |
| Educational | `bento-grid` + `chalkboard` |
| Journey | `winding-roadmap` + `storybook-watercolor` |
| Categories | `periodic-table` + `bold-graphic` |
| **High-density guide** | `dense-modules` + `morandi-journal` / `pop-laboratory` / `retro-pop-grid` |

### Keyword Shortcuts (Auto-select)
| User Keyword | Layout | Recommended Styles | Default Aspect | Prompt Notes |
|--------------|--------|-------------------|----------------|--------------|
| 高密度信息大图 / high-density-info | `dense-modules` | `morandi-journal`, `pop-laboratory`, `retro-pop-grid` | portrait | — |
| 信息图 / infographic | `bento-grid` | `craft-handmade` | landscape | Clean canvas, ample whitespace, simple cartoon elements |

### Output Structure
```
infographic/{topic-slug}/
├── source-{slug}.{ext}
├── analysis.md
├── structured-content.md
├── prompts/infographic.md
└── infographic.png
```

### Core Principles
- **Preserve source data faithfully** — no summarization/rephrasing (strip credentials/secrets)
- **Define learning objectives** before structuring
- **Structure for visual communication** (headlines, labels, visual elements)

### Workflow (7 Steps)
1. **Analyze Content** → `analysis.md`, `source-{slug}.md`
2. **Generate Structured Content** → `structured-content.md` (titles, sections, data points, design instructions)
3. **Recommend Combinations** — check keyword shortcuts first, then 3-5 layout×style recs
4. **Confirm Options** — `clarify` tool for combination, aspect ratio, language
5. **Generate Prompt** → `prompts/infographic.md` (combines layout def + style def + base template + content)
6. **Generate Image** — `image_generate` tool with mapped aspect ratio
7. **Output Summary** — topic, layout, style, aspect, language, output path

### References (from original skill)
- `references/analysis-framework.md`
- `references/structured-content-template.md`
- `references/base-prompt.md`
- `references/layouts/<layout>.md` (21 files)
- `references/styles/<style>.md` (21 files)

---

## Knowledge Comics

> **Source:** Absorbed from `baoyu-comic` skill. Knowledge comics (知识漫画): educational, biography, tutorial.

### When to Use
- Educational comics explaining concepts
- Biography/history comics
- Tutorial/how-to comics
- "知识漫画", "教育漫画", "Logicomix-style" requests

### Visual Dimensions
| Option | Values | Description |
|--------|--------|-------------|
| Art | `ligne-claire` (default), `manga`, `realistic`, `ink-brush`, `chalk`, `minimalist` | Art style |
| Tone | `neutral` (default), `warm`, `dramatic`, `romantic`, `energetic`, `vintage`, `action` | Mood |
| Layout | `standard` (default), `cinematic`, `dense`, `splash`, `mixed`, `webtoon`, `four-panel` | Panel arrangement |
| Aspect | `3:4` (default), `4:3`, `16:9` | Page ratio |
| Language | `auto` (default), `zh`, `en`, `ja` | Output language |

### Presets (5 with special rules)
| Preset | Equivalent | Hook |
|--------|-----------|------|
| `ohmsha` | manga + neutral | Visual metaphors, no talking heads, gadget reveals |
| `wuxia` | ink-brush + action | Qi effects, combat visuals, atmospheric |
| `shoujo` | manga + romantic | Decorative elements, eye details, romantic beats |
| `concept-story` | manga + warm | Visual symbol system, growth arc, dialogue+action balance |
| `four-panel` | minimalist + neutral + four-panel | 起承转合 structure, B&W + spot color, stick figures |

### File Structure
```
comic/{topic-slug}/
├── source-{slug}.md
├── analysis.md
├── storyboard.md
├── characters/characters.md
├── characters/characters.png
├── prompts/NN-{cover|page}-[slug].md
├── NN-{cover|page}-[slug].png
└── refs/NN-ref-{slug}.{ext}  (user-supplied reference images)
```

### Reference Images (Prompt-Only Extraction)
`image_generate` does NOT accept reference images. When user supplies refs:
1. Copy to `refs/NN-ref-{slug}.{ext}` for provenance
2. **Extract traits in text** → embed in every page prompt:
   - `style`: line treatment, texture, mood
   - `palette`: hex colors
   - `scene`: composition/subject notes
3. **Character consistency** via text descriptions in `characters/characters.md` (embedded in every page prompt)
4. Optional PNG character sheet (`characters/characters.png`) — human review only, not input to model

### Workflow (8 Steps with Progress Checklist)
```
Input → Analyze → [Check Existing?] → [Confirm: Style + Reviews] → Storyboard → [Review?] → Prompts → [Review?] → Images → Complete
```

**Step 1** — Analyze content → `analysis.md`, `source-{slug}.md`
**Step 2** — Confirm style, focus, audience, reviews (REQUIRED, use `clarify`)
**Step 3** — Generate storyboard + characters → `storyboard.md`, `characters/`
**Step 4** — Review outline (conditional)
**Step 5** — Generate prompts → `prompts/NN-{cover|page}-[slug].md`
**Step 6** — Review prompts (conditional)
**Step 7.1** — Character sheet (if multi-page + recurring chars) → `characters/characters.png`
**Step 7.2** — Generate pages (character descriptions embedded in every prompt)
**Step 8** — Completion report

**Critical:** `clarify` timeout → default for THAT question only, surface default visibly, continue with remaining questions.

### Image Generation (Step 7)
- Write each prompt to `prompts/NN-{type}-[slug].md` BEFORE calling `image_generate`
- Map aspect: `3:4`/`9:16`/`2:3` → `portrait`; `4:3`/`16:9`/`3:2` → `landscape`; `1:1` → `square`
- **Always download URL to absolute path**: `curl -fsSL "<url>" -o /abs/path/to/comic/{slug}/NN-page-{slug}.png`
- Backup existing: rename with `-backup-YYYYMMDD-HHMMSS`

### Page Modification
| Action | Steps |
|--------|-------|
| **Edit** | Update prompt file FIRST → regenerate → download new PNG |
| **Add** | Create prompt at position → generate → renumber subsequent → update storyboard |
| **Delete** | Remove files → renumber → update storyboard |

### Pitfalls
- Generation: 10-30s/page; auto-retry once on failure
- **Always download** URLs to local PNGs (not ephemeral URLs)
- **Absolute paths for `curl -o`** — never rely on persistent-shell CWD
- Use stylized alternatives for sensitive public figures
- **Step 2 confirmation REQUIRED** — do not skip
- **Strip secrets** — scan source for API keys/tokens before writing outputs

### References (from original skill)
- `references/base-prompt.md`
- `references/workflow.md`
- `references/storyboard-template.md`
- `references/ohmsha-guide.md`
- `references/partial-workflows.md`
- `references/auto-selection.md`
- `references/analysis-framework.md`
- `references/character-template.md`

---

## Decision Matrix

| Task | Section | Why |
|------|---------|-----|
| **Software/cloud architecture** | Architecture Diagrams | SVG/HTML, dark tech aesthetic, offline |
| **Data comparison, timeline, hierarchy, metrics** | Infographics | 21×21 combinations, structured workflow |
| **Educational explanation, biography, tutorial** | Knowledge Comics | Character consistency, panel layouts, presets |
| **Quick ASCII/text diagrams** | See `creative-ascii` | Terminal-native, no browser needed |
| **AI-generated artistic images** | See `creative-generative-ai` | ComfyUI/Stable Diffusion, open-ended |

---

## Related Skills

- **`creative-ascii`** — Terminal-native diagrams (ASCII, boxes, cowsay)
- **`creative-generative-ai`** — ComfyUI, Stable Diffusion for open-ended image gen
- **`creative-coding-animation`** — Manim, p5js for animated explanations
- **`creative-design-systems`** — Design system references (Stripe, Linear, Vercel styles)
- **`creative/humanizer`** — De-slop AI-generated text in diagrams/comics
