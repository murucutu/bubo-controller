# Variants & SVG Workflow

This document explains how variants work in Bubo Controller, the role of SVG blueprints, and the **3-track evolution strategy** for SVG integration.

---

## Table of Contents

- [Current state: Track C implemented](#current-state-track-c-implemented)
- [Current variants](#current-variants)
- [What is an SVG blueprint?](#what-is-an-svg-blueprint)
- [The 3-track evolution](#the-3-track-evolution)
- [Translating a new SVG to HTML (Track C)](#translating-a-new-svg-to-html-track-c)
- [Inkscape workflow for new variants](#inkscape-workflow-for-new-variants)
- [Naming conventions](#naming-conventions)

---

## Current state: Track C implemented

The project has migrated to **Track C** (SVG inline, JS-parsed). Both the Xbox and Arcade variants use a single `index.html` file with both SVGs inlined as `<template>` elements. JS clones the appropriate SVG based on `?controller=` and animates buttons directly.

This means:
- **Single HTML file** supports all controller layouts
- **SVG is the source of truth** — button positions come from the SVG, not from hardcoded CSS
- **Designer-friendly** — edit the SVG in Inkscape, re-inline into HTML, done
- **Zero drift** — SVG and HTML are the same thing

See [masks/controllers/README.md](masks/controllers/README.md) for the SVG structure requirements.

---

## Current variants

| File | Variant | Layout | SVG blueprint | Track |
| ---- | ------- | ------ | ------------- | ----- |
| `index.html?controller=xbox` | Xbox-style gamepad | Xbox Series layout | `masks/controllers/xbox.svg` | C |
| `index.html?controller=arcade` | Arcade fightstick | Viewlix layout | `masks/controllers/arcade.svg` | C |

Both variants share the same JS class (`BuboController`) and CSS. The SVGs are inlined as `<template>` elements in `index.html`. At runtime, JS clones the appropriate template, processes it (strips inline styles, adds labels), and animates buttons based on gamepad input.

---

## What is an SVG blueprint?

An SVG blueprint is a hand-drawn SVG file (typically created in Inkscape) that serves as both:
1. **Visual art** — what the user sees on stream
2. **Position reference** — where each button is located

Each button is a `<g id="BUTTON_ID">` with a `transform` attribute. JS finds these groups by ID and animates them (adds/removes `.pressed` class, moves knobs via CSS `transform`).

The SVGs live in `masks/controllers/` as the canonical source files. They are also inlined into `index.html` as `<template>` elements for runtime use (so the overlay works with `file://` protocol in OBS without needing `fetch()`).

---

## The 3-track evolution

The project has evolved through three tracks. We are now on Track C.

### Track A — Static blueprint (historical)

**How it worked**: SVG was a reference document. HTML had hardcoded CSS positions matching the SVG. The two files were kept in sync manually.

**Status**: ✅ Superseded by Track C.

### Track B — SVG as background (skipped)

**How it would work**: SVG replaces PNG as the controller background. Button positions still in HTML.

**Status**: ⏭️ Skipped in favor of Track C (which is more flexible).

### Track C — SVG inline, parsed (current)

**How it works**: SVG is inline in the HTML (as `<template>` elements). JS reads each `<g id="X">` and animates it directly. No CSS positions to maintain — the SVG IS the layout.

```
masks/controllers/xbox.svg     ←  drawn in Inkscape (source of truth)
       ↓ (inlined as <template>)
index.html                      ←  single file, both SVGs inline
  ├─ <template id="svg-xbox">    ←  Xbox SVG inline
  ├─ <template id="svg-arcade">  ←  Arcade SVG inline
  └─ <script>                    ←  JS clones + animates based on ?controller=
```

**Pros**:
- True single source of truth (SVG IS the HTML)
- Designer works in Inkscape, exports, maintainer inlines into HTML
- No drift possible
- Adding a new variant = add a new `<template>` + one line in the `CONTROLLERS` map

**Cons**:
- HTML file is larger (~55KB with both SVGs inline)
- Canvas-based effects (stick trail, trigger wave) need sibling `<canvas>` elements positioned via `getBBox()`

**Status**: ✅ Current implementation.

---

## Translating a new SVG to HTML (Track C)

When you have a new SVG blueprint with named layers:

### Step 1 — Place the SVG

Put the SVG file in `masks/controllers/your_variant.svg`.

### Step 2 — Inline into index.html

Read the SVG file content, strip the `<?xml>` declaration and `<!DOCTYPE>`, and paste the `<svg>...</svg>` into a `<template>` element in `index.html`:

```html
<template id="svg-your_variant">
    <!-- SVG content here (minus XML declaration) -->
</template>
```

### Step 3 — Register the controller

Add an entry to the `CONTROLLERS` map in the JS:

```js
const CONTROLLERS = Object.freeze({
    xbox: 'svg-xbox',
    arcade: 'svg-arcade',
    your_variant: 'svg-your_variant'  // new
});
```

### Step 4 — Test

Open `index.html?controller=your_variant` in a browser or OBS. The SVG should render, and buttons should light up when pressed.

### Automated inlining (for maintainers)

The project includes a Python script at `scripts/generate_index.py` that reads SVGs from `masks/controllers/` and generates the complete `index.html` with both SVGs inlined. To add a new variant:

1. Place the SVG in `masks/controllers/`.
2. Edit the script to read and inline the new SVG.
3. Add the new entry to the `CONTROLLERS` map in the script.
4. Run `python3 scripts/generate_index.py` to regenerate `index.html`.

---

## Inkscape workflow for new variants

1. **Set the document size** to 768×324 (matching the project's standard viewBox).
2. **Draw the background rect** (0,0 to 768,324).
3. **Draw each button** as a group with two paths (outer + inner).
4. **Rename each layer** to the controller element ID (`a`, `lb`, `left_joy`, etc.).
5. **Set `fill:white`** on the inner path of each button (will be stripped by JS at runtime).
6. **Export as Plain SVG** (not "Inkscape SVG").
7. **Verify** the exported SVG has `<g id="...">` elements matching your layer names.

See [masks/controllers/README.md](masks/controllers/README.md) for the full SVG structure requirements.

---

## Naming conventions

The project uses a consistent naming scheme for SVG layer IDs. Stick to it for new variants:

### Standard button names

| ID | Element |
| -- | ------- |
| `a`, `b`, `x`, `y` | Face buttons (ABXY slots) |
| `lb`, `rb` | Bumpers (shoulder buttons) |
| `lt`, `rt` | Triggers (analog) |
| `ls`, `rs` | Analog sticks (knob) — Xbox uses `left_joy`/`right_joy` |
| `left_joy_reach`, `right_joy_reach` | Analog stick boundaries (Xbox) |
| `joy`, `joy_reach` | Arcade stick + boundary (Arcade only) |
| `view`, `menu` | View/Menu (Xbox) or Share/Options (PS) or -/+ (Nintendo) |
| `up`, `down`, `left`, `right` | D-pad directions (Xbox only) |
| `l3`, `r3` | Stick-click buttons (Arcade: separate round buttons; Xbox: knob press) |

### Special suffixes

- `_reach` suffix: JS applies lower opacity (15%) — used for analog stick boundary rings.
- `Prancheta1`: the root group containing the background rect and all buttons (Inkscape convention).

---

## When to add a new variant

| Signal | Action |
| ------ | ------ |
| New controller layout (DualSense, Nintendo Pro, Steam Deck) | Create SVG in `masks/controllers/`, inline into `index.html`, add to `CONTROLLERS` map |
| Custom-themed overlay (neon, retro, branded) | Create a new SVG variant with custom art |
| Community contributes a layout | Same process — SVG + inline + register |

### Anti-patterns

- **Don't hardcode button positions in CSS** — let the SVG be the source of truth.
- **Don't use `fetch()` to load SVGs at runtime** — OBS `file://` protocol may block it. Always inline.
- **Don't mix tracks** — all variants should use Track C (inline SVG).
