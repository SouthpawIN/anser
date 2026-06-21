---
name: design-utilities
description: "Umbrella for creative skills: design-md, excalidraw, sketch"
author: Hermes Agent
version: 1.0.0
license: MIT
tags: ["Umbrella", "Creative", "Consolidated"]
related_skills: ["design-md", "excalidraw", "sketch"]
---

# Design Utilities

## DESIGN.md Skill


# DESIGN.md Skill

DESIGN.md is Google's open spec (Apache-2.0, `google-labs-code/design.md`) for
describing a visual identity to coding agents. One file combines:

- **YAML front matter** — machine-readable design tokens (normative values)
- **Markdown body** — human-readable rationale, organized into canonical sections

Tokens give exact values. Prose tells agents *why* those values exist and how to
apply them. The CLI (`npx @google/design.md`) lints structure + WCAG contrast,
diffs versions for regressions, and exports to Tailwind or W3C DTCG JSON.

## When to use this skill

- User asks for a DESIGN.md file, design tokens, or a design system spec
- User wants consistent UI/brand across multiple projects or tools
- User pastes an existing DESIGN.md and asks to lint, diff, export, or extend it
- User asks to port a style guide into a format agents can consume
- User wants contrast / WCAG accessibility validation on their color palette

For purely visual inspiration or layout examples, use `popular-web-designs`
instead. For *process and taste* when designing a one-off HTML artifact
from scratch (prototype, deck, landing page, component lab), use
`claude-design`. This skill is for the *formal spec file* itself.

## File anatomy

```md
---
version: alpha
name: Heritage
description: Architectural minimalism meets journalistic gravitas.
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"
  neutral: "#F7F5F2"
typography:
  h1:
    fontFamily: Public Sans
    fontSize: 3rem
    fontWeight: 700
    lineHeight: 1.1
    letterSpacing: "-0.02em"
  body-md:
    fontFamily: Public Sans
    fontSize: 1rem
rounded:
  sm: 4px
  md: 8px
  lg: 16px
spacing:
  sm: 8px
  md: 16px
  lg: 24px
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor: "#FFFFFF"
    rounded: "{rounded.sm}"
    padding: 12px
  button-primary-hover:
    backgroundColor: "{colors.primary}"
---

## Overview

Architectural Minimalism meets Journalistic Gravitas...

## Colors

- **Primary (#1A1C1E):** Deep ink for headlines and core text.
- **Tertiary (#B8422E):** "Boston Clay" — the sole driver for interaction.

## Typography

Public Sans for everything except small all-caps labels...

## Components

`button-primary` is the only high-emphasis action on a page...
```

## Token types

| Type | Format | Example |
|------|--------|---------|
| Color | `#` + hex (sRGB) | `"#1A1C1E"` |
| Dimension | number + unit (`px`, `em`, `rem`) | `48px`, `-0.02em` |
| Token reference | `{path.to.token}` | `{colors.primary}` |
| Typography | object with `fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing`, `fontFeature`, `fontVariation` | see above |

Component property whitelist: `backgroundColor`, `textColor`, `typography`,
`rounded`, `padding`, `size`, `height`, `width`. Variants (hover, active,
pressed) are **separate component entries** with related key names
(`button-primary-hover`), not nested.

## Canonical section order

Sections are optional, but present ones MUST appear in this order. Duplicate
headings reject the file.

1. Overview (alias: Brand & Style)
2. Colors
3. Typography
4. Layout (alias: Layout & Spacing)
5. Elevation & Depth (alias: Elevation)
6. Shapes
7. Components
8. Do's and Don'ts

Unknown sections are preserved, not errored. Unknown token names are accepted
if the value type is valid. Unknown component properties produce a warning.

## Workflow: authoring a new DESIGN.md

1. **Ask the user** (or infer) the brand tone, accent color, and typography
   direction. If they provided a site, image, or vibe, translate it to the
   token shape above.
2. **Write `DESIGN.md`** in their project root using `write_file`. Always
   include `name:` and `colors:`; other sections optional but encouraged.
3. **Use token references** (`{colors.primary}`) in the `components:` section
   instead of re-typing hex values. Keeps the palette single-source.
4. **Lint it** (see below). Fix any broken references or WCAG failures
   before returning.
5. **If the user has an existing project**, also write Tailwind or DTCG
   exports next to the file (`tailwind.theme.json`, `tokens.json`).

## Workflow: lint / diff / export

The CLI is `@google/design.md` (Node). Use `npx` — no global install needed.

```bash
# Validate structure + token references + WCAG contrast
npx -y @google/design.md lint DESIGN.md

# Compare two versions, fail on regression (exit 1 = regression)
npx -y @google/design.md diff DESIGN.md DESIGN-v2.md

# Export to Tailwind theme JSON
npx -y @google/design.md export --format tailwind DESIGN.md > tailwind.theme.json

# Export to W3C DTCG (Design Tokens Format Module) JSON
npx -y @google/design.md export --format dtcg DESIGN.md > tokens.json

# Print the spec itself — useful when injecting into an agent prompt
npx -y @google/design.md spec --rules-only --format json
```

All commands accept `-` for stdin. `lint` returns exit 1 on errors. Use the
`--format json` flag and parse the output if you need to report findings
structurally.

### Lint rule reference (what the 7 rules catch)

- `broken-ref` (error) — `{colors.missing}` points at a non-existent token
- `duplicate-section` (error) — same `## Heading` appears twice
- `invalid-color`, `invalid-dimension`, `invalid-typography` (error)
- `wcag-contrast` (warning/info) — component `textColor` vs `backgroundColor`
  ratio against WCAG AA (4.5:1) and AAA (7:1)
- `unknown-component-property` (warning) — outside the whitelist above

When the user cares about accessibility, call this out explicitly in your
summary — WCAG findings are the most load-bearing reason to use the CLI.

## Pitfalls

- **Don't nest component variants.** `button-primary.hover` is wrong;
  `button-primary-hover` as a sibling key is right.
- **Hex colors must be quoted strings.** YAML will otherwise choke on `#` or
  truncate values like `#1A1C1E` oddly.
- **Negative dimensions need quotes too.** `letterSpacing: -0.02em` parses as
  a YAML flow — write `letterSpacing: "-0.02em"`.
- **Section order is enforced.** If the user gives you prose in a random order,
  reorder it to match the canonical list before saving.
- **`version: alpha` is the current spec version** (as of Apr 2026). The spec
  is marked alpha — watch for breaking changes.
- **Token references resolve by dotted path.** `{colors.primary}` works;
  `{primary}` does not.

## Spec source of truth

- Repo: https://github.com/google-labs-code/design.md (Apache-2.0)
- CLI: `@google/design.md` on npm
- License of generated DESIGN.md files: whatever the user's project uses;
  the spec itself is Apache-2.0.


## Excalidraw Diagram Skill


# Excalidraw Diagram Skill

Create diagrams by writing standard Excalidraw element JSON and saving as `.excalidraw` files. These files can be drag-and-dropped onto [excalidraw.com](https://excalidraw.com) for viewing and editing. No accounts, no API keys, no rendering libraries -- just JSON.

## When to use

Generate `.excalidraw` files for architecture diagrams, flowcharts, sequence diagrams, concept maps, and more. Files can be opened at excalidraw.com or uploaded for shareable links.

## Workflow

1. **Load this skill** (you already did)
2. **Write the elements JSON** -- an array of Excalidraw element objects
3. **Save the file** using `write_file` to create a `.excalidraw` file
4. **Optionally upload** for a shareable link using `scripts/upload.py` via `terminal`

### Saving a Diagram

Wrap your elements array in the standard `.excalidraw` envelope and save with `write_file`:

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "hermes-agent",
  "elements": [ ...your elements array here... ],
  "appState": {
    "viewBackgroundColor": "#ffffff"
  }
}
```

Save to any path, e.g. `~/diagrams/my_diagram.excalidraw`.

### Uploading for a Shareable Link

Run the upload script (located in this skill's `scripts/` directory) via terminal:

```bash
python skills/diagramming/excalidraw/scripts/upload.py ~/diagrams/my_diagram.excalidraw
```

This uploads to excalidraw.com (no account needed) and prints a shareable URL. Requires the `cryptography` pip package (`pip install cryptography`).

---

## Element Format Reference

### Required Fields (all elements)
`type`, `id` (unique string), `x`, `y`, `width`, `height`

### Defaults (skip these -- they're applied automatically)
- `strokeColor`: `"#1e1e1e"`
- `backgroundColor`: `"transparent"`
- `fillStyle`: `"solid"`
- `strokeWidth`: `2`
- `roughness`: `1` (hand-drawn look)
- `opacity`: `100`

Canvas background is white.

### Element Types

**Rectangle**:
```json
{ "type": "rectangle", "id": "r1", "x": 100, "y": 100, "width": 200, "height": 100 }
```
- `roundness: { "type": 3 }` for rounded corners
- `backgroundColor: "#a5d8ff"`, `fillStyle: "solid"` for filled

**Ellipse**:
```json
{ "type": "ellipse", "id": "e1", "x": 100, "y": 100, "width": 150, "height": 150 }
```

**Diamond**:
```json
{ "type": "diamond", "id": "d1", "x": 100, "y": 100, "width": 150, "height": 150 }
```

**Labeled shape (container binding)** -- create a text element bound to the shape:

> **WARNING:** Do NOT use `"label": { "text": "..." }` on shapes. This is NOT a valid
> Excalidraw property and will be silently ignored, producing blank shapes. You MUST
> use the container binding approach below.

The shape needs `boundElements` listing the text, and the text needs `containerId` pointing back:
```json
{ "type": "rectangle", "id": "r1", "x": 100, "y": 100, "width": 200, "height": 80,
  "roundness": { "type": 3 }, "backgroundColor": "#a5d8ff", "fillStyle": "solid",
  "boundElements": [{ "id": "t_r1", "type": "text" }] },
{ "type": "text", "id": "t_r1", "x": 105, "y": 110, "width": 190, "height": 25,
  "text": "Hello", "fontSize": 20, "fontFamily": 1, "strokeColor": "#1e1e1e",
  "textAlign": "center", "verticalAlign": "middle",
  "containerId": "r1", "originalText": "Hello", "autoResize": true }
```
- Works on rectangle, ellipse, diamond
- Text is auto-centered by Excalidraw when `containerId` is set
- The text `x`/`y`/`width`/`height` are approximate -- Excalidraw recalculates them on load
- `originalText` should match `text`
- Always include `fontFamily: 1` (Virgil/hand-drawn font)

**Labeled arrow** -- same container binding approach:
```json
{ "type": "arrow", "id": "a1", "x": 300, "y": 150, "width": 200, "height": 0,
  "points": [[0,0],[200,0]], "endArrowhead": "arrow",
  "boundElements": [{ "id": "t_a1", "type": "text" }] },
{ "type": "text", "id": "t_a1", "x": 370, "y": 130, "width": 60, "height": 20,
  "text": "connects", "fontSize": 16, "fontFamily": 1, "strokeColor": "#1e1e1e",
  "textAlign": "center", "verticalAlign": "middle",
  "containerId": "a1", "originalText": "connects", "autoResize": true }
```

**Standalone text** (titles and annotations only -- no container):
```json
{ "type": "text", "id": "t1", "x": 150, "y": 138, "text": "Hello", "fontSize": 20,
  "fontFamily": 1, "strokeColor": "#1e1e1e", "originalText": "Hello", "autoResize": true }
```
- `x` is the LEFT edge. To center at position `cx`: `x = cx - (text.length * fontSize * 0.5) / 2`
- Do NOT rely on `textAlign` or `width` for positioning

**Arrow**:
```json
{ "type": "arrow", "id": "a1", "x": 300, "y": 150, "width": 200, "height": 0,
  "points": [[0,0],[200,0]], "endArrowhead": "arrow" }
```
- `points`: `[dx, dy]` offsets from element `x`, `y`
- `endArrowhead`: `null` | `"arrow"` | `"bar"` | `"dot"` | `"triangle"`
- `strokeStyle`: `"solid"` (default) | `"dashed"` | `"dotted"`

### Arrow Bindings (connect arrows to shapes)

```json
{
  "type": "arrow", "id": "a1", "x": 300, "y": 150, "width": 150, "height": 0,
  "points": [[0,0],[150,0]], "endArrowhead": "arrow",
  "startBinding": { "elementId": "r1", "fixedPoint": [1, 0.5] },
  "endBinding": { "elementId": "r2", "fixedPoint": [0, 0.5] }
}
```

`fixedPoint` coordinates: `top=[0.5,0]`, `bottom=[0.5,1]`, `left=[0,0.5]`, `right=[1,0.5]`

### Drawing Order (z-order)
- Array order = z-order (first = back, last = front)
- Emit progressively: background zones → shape → its bound text → its arrows → next shape
- BAD: all rectangles, then all texts, then all arrows
- GOOD: bg_zone → shape1 → text_for_shape1 → arrow1 → arrow_label_text → shape2 → text_for_shape2 → ...
- Always place the bound text element immediately after its container shape

### Sizing Guidelines

**Font sizes:**
- Minimum `fontSize`: **16** for body text, labels, descriptions
- Minimum `fontSize`: **20** for titles and headings
- Minimum `fontSize`: **14** for secondary annotations only (sparingly)
- NEVER use `fontSize` below 14

**Element sizes:**
- Minimum shape size: 120x60 for labeled rectangles/ellipses
- Leave 20-30px gaps between elements minimum
- Prefer fewer, larger elements over many tiny ones

### Color Palette

See `references/colors.md` for full color tables. Quick reference:

| Use | Fill Color | Hex |
|-----|-----------|-----|
| Primary / Input | Light Blue | `#a5d8ff` |
| Success / Output | Light Green | `#b2f2bb` |
| Warning / External | Light Orange | `#ffd8a8` |
| Processing / Special | Light Purple | `#d0bfff` |
| Error / Critical | Light Red | `#ffc9c9` |
| Notes / Decisions | Light Yellow | `#fff3bf` |
| Storage / Data | Light Teal | `#c3fae8` |

### Tips
- Use the color palette consistently across the diagram
- **Text contrast is CRITICAL** -- never use light gray on white backgrounds. Minimum text color on white: `#757575`
- Do NOT use emoji in text -- they don't render in Excalidraw's font
- For dark mode diagrams, see `references/dark-mode.md`
- For larger examples, see `references/examples.md`




## Sketch


# Sketch

Use this skill when the user wants to **see a design direction before committing** to one — exploring a UI/UX idea as disposable HTML mockups. The point is to generate 2-3 interactive variants so the user can compare visual directions side-by-side, not to produce shippable code.

Load this when the user says things like "sketch this screen", "show me what X could look like", "compare layout A vs B", "give me 2-3 takes on this UI", "let me see some variants", "mockup this before I build".

## When NOT to use this

- User wants a production component — use `claude-design` or build it properly
- User wants a polished one-off HTML artifact (landing page, deck) — `claude-design`
- User wants a diagram — `excalidraw`, `architecture-diagram`
- The design is already locked — just build it

## If the user has the full GSD system installed

If `gsd-sketch` shows up as a sibling skill (installed via `npx get-shit-done-cc --hermes`), prefer **`gsd-sketch`** for the full workflow: persistent `.planning/sketches/` with MANIFEST, frontier mode analysis, consistency audits across past sketches, and integration with the rest of GSD. This skill is the lightweight standalone version — one-off sketching without the state machinery.

## Core method

```
intake  →  variants  →  head-to-head  →  pick winner (or iterate)
```

### 1. Intake (skip if the user already gave you enough)

Before generating variants, get three things — one question at a time, not all at once:

1. **Feel.** "What should this feel like? Adjectives, emotions, a vibe." — *"calm, editorial, like Linear"* tells you more than *"minimal"*.
2. **References.** "What apps, sites, or products capture the feel you're imagining?" — actual references beat abstract descriptions.
3. **Core action.** "What's the single most important thing a user does on this screen?" — the variants should all serve this well; if they don't, they're just decoration.

Reflect each answer briefly before the next question. If the user already gave you all three upfront, skip straight to variants.

### 2. Variants (2-3, never 1, rarely 4+)

Produce **2-3 variants** in one go. Each variant is a complete, standalone HTML file. Don't describe variants — build them. The point is comparison.

Each variant should take a **different design stance**, not different pixel values. Three good variant axes:

- **Density:** compact / airy / ultra-dense (pick two contrasting poles)
- **Emphasis:** content-first / action-first / tool-first
- **Aesthetic:** editorial / utilitarian / playful
- **Layout:** single-column / sidebar / split-pane
- **Grounding:** card-based / bare-content / document-style

Pick one axis and pull apart from it. Two variants that differ only in accent color are wasted effort — the user can't distinguish them.

**Variant naming:** describe the stance, not the number.

```
sketches/
├── 001-calm-editorial/
│   ├── index.html
│   └── README.md
├── 001-utilitarian-dense/
│   ├── index.html
│   └── README.md
└── 001-playful-split/
    ├── index.html
    └── README.md
```

### 3. Make them real HTML

Each variant is a **single self-contained HTML file**:

- Inline `<style>` — no build step, no external CSS
- System fonts or one Google Font via `<link>`
- Tailwind via CDN (`<script src="https://cdn.tailwindcss.com"></script>`) is fine
- Realistic fake content — actual sentences, actual names, not "Lorem ipsum"
- **Interactive**: links clickable, hovers real, at least one state transition (open/close, filter, toggle). A frozen static image is a worse spike than a sloppy animated one.

Open it in a browser. If it looks broken, fix it before showing the user.

**Verify variants visually — use Hermes' browser tools.** Don't just write HTML and hope it renders; load each variant and look at it:

```
browser_navigate(url="file:///absolute/path/to/sketches/001-calm-editorial/index.html")
browser_vision(question="Does this layout look clean and readable? Any visible bugs (overlapping text, unstyled elements, broken images)?")
```

`browser_vision` returns an AI description of what's actually on the page plus a screenshot path — catches layout bugs that pure source inspection misses (e.g. a font import that silently failed, a flex container that collapsed). Fix and re-navigate until each variant looks right.

**Default CSS reset + system font stack** for fast starts:

```html
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
                 "Helvetica Neue", Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    color: #1a1a1a;
    background: #fafafa;
    line-height: 1.5;
  }
</style>
```

### 4. Variant README

Each variant's `README.md` answers:

```markdown
## Variant: {stance name}

### Design stance
One sentence on the principle driving this variant.

### Key choices
- Layout: ...
- Typography: ...
- Color: ...
- Interaction: ...

### Trade-offs
- Strong at: ...
- Weak at: ...

### Best for
- The kind of user or use case this variant actually serves
```

### 5. Head-to-head

After all variants are built, present them as a comparison. Don't just list — **opinionate**:

```markdown
## Three takes on the home screen

| Dimension | Calm editorial | Utilitarian dense | Playful split |
|-----------|----------------|-------------------|---------------|
| Density   | Low            | High              | Medium        |
| Primary action visibility | Low | High | Medium |
| Scan-ability | High | Medium | Low |
| Feel | Calm, trusted | Sharp, tool-like | Inviting, energetic |

**My take:** Utilitarian dense for power users, calm editorial for content-forward audiences. Playful split is weakest — tries to do both and commits to neither.
```

Let the user pick a winner, or combine two into a hybrid, or ask for another round.

## Theming (when the project has a visual identity)

If the user has an existing theme (colors, fonts, tokens), put shared tokens in `sketches/themes/tokens.css` and `@import` them in each variant. Keep tokens minimal:

```css
/* sketches/themes/tokens.css */
:root {
  --color-bg: #fafafa;
  --color-fg: #1a1a1a;
  --color-accent: #0066ff;
  --color-muted: #666;
  --radius: 8px;
  --font-display: "Inter", sans-serif;
  --font-body: -apple-system, BlinkMacSystemFont, sans-serif;
}
```

Don't over-tokenize a throwaway sketch — three colors and one font is usually enough.

## Interactivity bar

A sketch is interactive enough when the user can:

1. **Click a primary action** and something visible happens (state change, modal, toast, navigation feint)
2. **See one meaningful state transition** (filter a list, toggle a mode, open/close a panel)
3. **Hover recognizable affordances** (buttons, rows, tabs)

More than that is over-engineering a throwaway. Less than that is a screenshot.

## Frontier mode (picking what to sketch next)

If sketches already exist and the user says "what should I sketch next?":

- **Consistency gaps** — two winning variants from different sketches made independent choices that haven't been composed together yet
- **Unsketched screens** — referenced but never explored
- **State coverage** — happy path sketched, but not empty / loading / error / 1000-items
- **Responsive gaps** — validated at one viewport; does it hold at mobile / ultrawide?
- **Interaction patterns** — static layouts exist; transitions, drag, scroll behavior don't

Propose 2-4 named candidates. Let the user pick.

## Output

- Create `sketches/` (or `.planning/sketches/` if the user is using GSD conventions) in the repo root
- One subdir per variant: `NNN-stance-name/index.html` + `README.md`
- Tell the user how to open them: `open sketches/001-calm-editorial/index.html` on macOS, `xdg-open` on Linux, `start` on Windows
- Keep variants disposable — a sketch that you felt the need to preserve should be promoted into real project code, not curated as an asset

**Typical tool sequence for one variant:**

```
terminal("mkdir -p sketches/001-calm-editorial")
write_file("sketches/001-calm-editorial/index.html", "<!doctype html>...")
write_file("sketches/001-calm-editorial/README.md", "## Variant: Calm editorial\n...")
browser_navigate(url="file://$(pwd)/sketches/001-calm-editorial/index.html")
browser_vision(question="How does this look? Any obvious layout issues?")
```

Repeat for each variant, then present the comparison table.

## Attribution

Adapted from the GSD (Get Shit Done) project's `/gsd-sketch` workflow — MIT © 2025 Lex Christopherson ([gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done)). The full GSD system ships persistent sketch state, theme/variant pattern references, and consistency-audit workflows; install with `npx get-shit-done-cc --hermes --global`.

