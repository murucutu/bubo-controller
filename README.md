# Bubo Controller

**An owl-eyed gamepad overlay for OBS Browser Source.**

[![License: Unlicense](https://img.shields.io/badge/license-Unlicense-blue.svg)](LICENSE)
[![Made for OBS](https://img.shields.io/badge/made%20for-OBS-9146FF.svg)](https://obsproject.com/)
![Version](https://img.shields.io/badge/version-v1.2611.002-9146FF.svg)

**Version:** `v1.2611.002` · **Scheme:** `v1.YYMM.XXX` (v1 = published · YYMM = build month · XXX = sprint number, resets monthly) — see [CHANGELOG.md](CHANGELOG.md).

---

## About

Bubo Controller is a single-file HTML overlay that reads the browser's [Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API) and draws a gamepad on your OBS scene in real time. It supports multiple controller layouts (Xbox, Arcade fightstick) via swappable SVG blueprints. Triggers are pressure-sensitive, analog sticks are calibrated, and every button color is individually customizable.

**Why "Bubo"?** *Bubo* is the Latin name for the horned owl and the owl that accompanied Minerva, the Roman goddess of wisdom. The owl has watched over streamers since ancient times — and now it watches your gamepad. 🦉

The project is dedicated to the **public domain** ([The Unlicense](LICENSE)) — use it, fork it, sell it, derive from it. No attribution required.

---

## Features

- ✅ Single-file HTML — no build, no dependencies, no external requests
- ✅ **Multiple controller layouts** via SVG blueprints (`?controller=xbox|arcade`), both on a 640×216 canvas
- ✅ Standard Gamepad API (works with Xbox, PlayStation, generic controllers)
- ✅ Pressure-sensitive triggers LT/RT with reactive waveform graph, masked to the trigger contour (Xbox layout)
- ✅ Calibrated analog sticks with dead zone and visible reach rings
- ✅ **Arcade layout v2** with octagonal gate and ghost-glow stick trail in a fixed Nox→Sol gradient (purple `#8865F1` → pink `#F165A4`)
- ✅ Per-button colors via CSS variables (`--color-a`, `--color-b`, …)
- ✅ **ABXY color schemes** via URL (`?color=xbox|playstation`)
- ✅ Button labels swappable via URL (`?icons=xbox|playstation|nintendo|none`) or custom text
- ✅ **Gamepad selection** for multi-controller setups — stable across boots via USB id matching (`?gamepad=hori`)
- ✅ Bilingual "connect a controller" hint, positioned in the bottom gap (never overlaps the controller)
- ✅ Auto-scales to any OBS Browser Source size while preserving aspect ratio
- ✅ Optional fit modes (`?fit=contain|cover`) and manual scale (`?scale=2`)
- ✅ Debug mode (double-click the overlay) for visual inspection

---

## Quick Start

1. **Download & unzip** the project (or `git clone https://github.com/murucutu/bubo-controller.git`).
2. In OBS Studio, add a **Browser** source.
3. Check **Local file** and select `index.html`.
4. Set **Width** and **Height**. For full-bleed (no transparent padding), use any ~2.96:1 size (both layout canvases are 640×216):
   - `640 × 216` (base, scale 1×)
   - `1280 × 432` (2× — HD horizontal full-bleed)
   - `1920 × 648` (3× — Full HD horizontal full-bleed)
   - `3840 × 1296` (6× — 4K horizontal full-bleed)
   - Or any 16:9 (`1920×1080`, `3840×2160`) — the overlay fills the width, with vertical padding.
5. Plug in a gamepad. Press any button. The overlay reacts in real time.

> Tip: enable **"Shutdown source when not visible"** to save resources when the scene is inactive.

---

## URL Parameters

| Parameter | Values | Description |
| --------- | ------ | ----------- |
| `?controller=` | `xbox` (default) / `arcade` | Controller layout to display |
| `?color=` | `xbox` / `playstation` | ABXY color scheme — colors BOTH the outline AND the pressed fill (default: all Nimbus/white) |
| `?icons=` | `xbox` (default) / `playstation` / `nintendo` / `none` | Button label text set (ABXY, LB/RB, LT/RT, L3/R3) |
| `?viewmenu=` | `1` / `true` / `yes` / `on` | Show the VIEW/MENU icons (two-squares for View, hamburger for Menu). Hidden by default. |
| `?scale=X` | any positive number | Manual scale override (e.g. `?scale=2` = 2× zoom) |
| `?fit=` | `contain` (default) / `cover` | How the overlay fits the source: contain = preserve aspect with padding; cover = fill the source, may clip the sides |
| `?gamepad=` | `<N>` / `<id substring>` / `list` | Which gamepad to read when several are connected. `N` = 0-based index (**unstable**: follows the OS USB enumeration order, which can be re-shuffled between boots — one day index 1 is the fightstick, next day it's the Xbox pad). `<id substring>` = case-insensitive match against the USB id string reported by the browser (e.g. `Vendor: 0x0f0d Product: 0x00b1`) — **stable across boots** (e.g. `?gamepad=hori`, `?gamepad=fight`, `?gamepad=0x0f0d`). `list` = diagnostic mode: logs connected gamepads (index → id) to the console whenever the set changes. Out-of-range index falls back to index 0 (keeps the overlay alive). |

Combine freely: `?controller=xbox&color=xbox&viewmenu=1&scale=2`

> **Multiple controllers tip:** prefer `?gamepad=<id substring>` over `?gamepad=N`. The numeric index follows USB enumeration order, which the OS may reorder between boots. Open the Browser Source console once with `?gamepad=list` to see each pad's id, then pin the selection with a stable substring.

---

## Customization

All configuration lives in `:root` at the top of the `<style>` block in `index.html`. No build step — edit and reload the page in OBS.

### Global colors

Two semantic variables control the base theme, with two legacy aliases for backwards compatibility:

```css
:root {
    --nimbus: #F2F2F2;     /* outline color (buttons, knobs) — light gray */
    --umbral: #0D0D0D;     /* fill color (background, button fill, label text) — near-black */
    --accent: var(--nimbus);   /* alias — outlines (legacy variable name) */
    --color-bg: var(--umbral); /* alias — fills + label text contrast */
}
```

To invert to a light theme (Umbral outlines on a light background):

```css
:root {
    --nimbus: #0D0D0D;
    --umbral: #FFFFFF;
}
```

(The aliases follow automatically since they resolve to `--nimbus` and `--umbral`.)

### ABXY color schemes (`?color=`)

Swap the ABXY color set via URL — no HTML editing needed. Colors apply to button **outlines** and to the pressed fill.

| URL | A | B | X | Y |
| --- | --- | --- | --- | --- |
| `index.html` (default) | white | white | white | white |
| `?color=xbox` | green `#4CAF50` | red `#E5342A` | blue `#0E7DC0` | amber `#F2C200` |
| `?color=playstation` | blue `#2E6DB4` (✕ Cross) | red `#E5342A` (○ Circle) | pink `#FF69F8` (□ Square) | green `#3EE3A1` (△ Triangle) |

**Official color sources:**

- **Xbox** (Xbox 360 / Xbox One controller colors): A green `#4CAF50`, B red `#E5342A`, X blue `#0E7DC0`, Y amber `#F2C200`.
- **PlayStation** (per PS1/PS4 designer Teiyu Goto's published rationale): Cross blue ("yes"), Circle red ("no"), Square pink (a piece of paper / menu items), Triangle green (viewpoint / head). Consistent across PS1–PS5.

The `?color=` parameter only sets the 4 face button colors. All other buttons (LB, RB, LT, RT, D-pad, sticks) remain `--accent`. The arcade trail does **not** follow `?color=` — it always uses the Nox→Sol gradient.

### Per-button colors

Each button has its own CSS variable, defaulting to `--accent`:

| Button(s) | CSS variable |
| --------- | ------------ |
| A, B, X, Y | `--color-a`, `--color-b`, `--color-x`, `--color-y` |
| LB, RB (bumpers) | `--color-lb`, `--color-rb` |
| LT, RT (triggers) | `--color-lt`, `--color-rt` |
| LS, RS (analog sticks) | `--color-ls`, `--color-rs` |
| Arcade stick | `--color-stick` |
| D-pad (4 directions) | `--color-dpad` (one color for all 4) |
| View | `--color-view` |
| Menu | `--color-menu` |
| L3, R3 (stick click) | `--color-l3`, `--color-r3` — mapped to the Xbox stick knobs (`left_joy`/`right_joy`) |

#### Example — custom neon theme

```css
:root {
    --accent: #00FF7F;          /* neon green */
    --color-lt: #FF00FF;        /* magenta on LT */
    --color-rt: #00FFFF;        /* cyan on RT */
    --color-a: #FFD700;         /* gold on A */
}
```

> **Note**: `?color=` in the URL overrides `--color-a` through `--color-y` at load time. To use custom CSS colors for ABXY, don't use `?color=` in the URL.

### Opacity system

The overlay uses a layered opacity system with sensible defaults for OBS visibility:

```css
:root {
    --bg-opacity: 0.95;              /* background fill opacity (95%) */
    --button-fill-opacity: 0;        /* unpressed button fill (0% — outline-only idle) */
    --button-pressed-opacity: 1;     /* pressed button fill (100% solid) */
    --knob-fill-opacity: 0.5;        /* L3/R3 knob base (50% Nimbus) */
    --knob-pressed-opacity: 1;       /* L3/R3 knob pressed (100% Nimbus) */
}
```

**How it works:**

1. **Background** (640×216 canvas): filled with `--umbral` at 95% opacity — near-opaque, readable on any stream.
2. **Buttons in idle**: outline-only (fill opacity 0). The outline is `--nimbus` (or the active `?color=` scheme color) at 100%.
3. **Pressed buttons**: the fill color switches to the outline color (Nimbus by default, or the scheme color) at 100% opacity — a clean, visible "flash" on press.
4. **L3/R3 knobs** (Xbox layout): the knob fill is `--nimbus` at 50% as the base state (stick-click indicator); jumps to 100% when L3/R3 is pressed. Knobs keep Nimbus even when `?color=xbox` is active.
5. **Boundary rings** (`*_reach`): outline only, no fill — subtle, doesn't compete with buttons.

**Tuning** — for a lighter overlay:

```css
:root {
    --bg-opacity: 0.5;
    --button-fill-opacity: 0.15;   /* faint idle fill instead of outline-only */
}
```

### LT/RT trigger wave (Xbox only)

LT and RT have a gradient fill that grows from inside-out based on pressure, plus a canvas waveform overlay that reacts in real time — **masked to the trigger contour** via `clip-path: inset(0 round var(--trigger-clip-radius))` (default `9px`), so the wave never leaks outside the trigger outline. The wave color follows `--color-lt` / `--color-rt`. This effect is not affected by the opacity system (it has its own alpha).

### Button labels (`?icons=`)

| URL | ABXY labels | Shoulder labels |
| --- | --- | --- |
| `?icons=xbox` (default) | A B X Y | LB RB LT RT |
| `?icons=playstation` | ✕ ○ □ △ | L1 R1 L2 R2 |
| `?icons=nintendo` | B A Y X (Switch layout) | L R ZL ZR |
| `?icons=none` | (no labels) | (no labels) |

Labels are rendered as SVG `<text>` elements, positioned at the center of each button by JS. The label text color uses `--color-bg` for contrast.

**Custom labels** — edit the `--label-*` variables:

```css
:root {
    --label-a: "JUMP";
    --label-b: "SHOOT";
    --label-x: "GUARD";
    --label-y: "DASH";
}
```

**Label font and size:**

```css
:root {
    --label-font: "Inter", "Segoe UI", system-ui, sans-serif;
    --label-size: 18px;           /* ABXY */
    --label-size-small: 13px;     /* LB/RB/LT/RT/L3/R3 */
    --label-weight: 700;
}
```

### Arcade stick trail

The arcade layout features a ghost-glow stick trail: a fading ribbon that follows the knob, drawn on a canvas above the SVG.

**Trail colors — fixed Nox→Sol gradient.** The trail always renders a 4-stop gradient between Nox (purple `#8865F1`) at the newest position and Sol (pink `#F165A4`) at the oldest, with interpolated stops in between:

```css
:root {
    --nox: #8865F1;
    --sol: #F165A4;
    --trail-color-1: var(--nox);   /* newest, strongest */
    --trail-color-2: #AB65D9;      /* ~1/3 through the gradient */
    --trail-color-3: #CF65BD;      /* ~2/3 through the gradient */
    --trail-color-4: var(--sol);   /* oldest, faintest */
}
```

The gradient does not follow `?color=` schemes. To use a custom gradient, set the four `--trail-color-*` variables to your own colors (keep `--trail-color-1` the strongest and `--trail-color-4` the faintest — alpha is applied per-position automatically).

**Trail parameters:**

```css
:root {
    --stick-trail-length: 60;     /* number of trail frames (~1s at 60fps) */
    --stick-trail-fade: 0.04;     /* fade rate per frame (lower = softer) */
    --stick-trail-radius: 8;      /* trail circle radius (1/3 of the 24-unit knob) */
    --stick-trail-blur: 12px;     /* ghost glow blur */
}
```

**Tuning examples:**

```css
/* Softer, longer trail (more ethereal) */
:root {
    --stick-trail-length: 90;
    --stick-trail-fade: 0.02;
    --stick-trail-blur: 16px;
}

/* Sharper, shorter trail (more precise) */
:root {
    --stick-trail-length: 25;
    --stick-trail-fade: 0.10;
    --stick-trail-blur: 6px;
}
```

**How the trail works:** each animation frame records the knob position; the trail stores the last N positions (N = length) and draws connecting line segments with per-segment interpolated color + alpha (continuous, no gaps), plus circles at ⅓ of the knob size with a neon glow on the older positions. The knob itself renders white in front, with a 3px glow ring.

**Analog + D-pad capture:** the arcade stick captures both analog axes (axes 0, 1) and D-pad buttons (12–15) simultaneously — if any D-pad direction is pressed, it overrides the analog position for that axis, so fightsticks that report either analog or digital input work seamlessly.

### "No controller" hint

A subtle bilingual hint ("Connect a controller" / "Conecte um controle") shows after 5 seconds without a gamepad, hides when one connects, and never reappears after the first connection (avoids mid-stream noise). It sits in the bottom 27px gap of the canvas — never overlapping the controller body:

```css
:root {
    --hint-color: var(--nimbus);
    --hint-opacity: 0.6;
    --hint-font-weight: 900;
    --hint-size: 14px;
    --hint-delay-ms: 5000;        /* delay before showing */
    --hint-transition-ms: 200;    /* fade-in duration */
    --hint-height: 18px;          /* text band height */
    --hint-bottom-gap: 4.5px;     /* margin from the bottom edge */
}
```

The hint uses the system font stack at weight 900 (no external fonts — the overlay must work offline). Double-click the overlay (debug mode) to force-show the hint for visual QA.

---

## Adding a new controller layout

Layouts are SVG blueprints: the SVG is both the **art** (what the user sees) and the **position reference** (where each button is). The runtime clones the SVG, animates the buttons via the Gamepad API, and CSS controls all colors and states.

### How rendering works

1. The SVG in `masks/controllers/<layout>.svg` is the **source of truth** for that layout's button positions, sizes, and shapes.
2. The SVG is inlined into `index.html` as a `<template id="svg-<layout>">` (so the overlay works from `file://` in OBS without runtime `fetch()`).
3. At runtime, JS clones the appropriate template based on `?controller=`, processes it (splits ring paths into fill + outline, adds labels), and animates each `<g id="<button>">`.
4. CSS controls all fills, opacities, and pressed states via the `--color-*` variables in `:root`.

### SVG structure requirements

```xml
<svg viewBox="0 0 640 216" xmlns="http://www.w3.org/2000/svg">
    <g id="<root>">
        <!-- Background (640×216) — fill controlled by CSS -->
        <path d="..." />

        <!-- Each button is a <g> with a unique id -->
        <g id="a" transform="matrix(...)">
            <!-- Ring path: outer + inner subpaths (fill-rule: evenodd renders as outline) -->
            <path d="..." />
        </g>

        <!-- ... more buttons ... -->
    </g>
</svg>
```

1. **ViewBox**: `0 0 640 216` (mandatory — matches the overlay dimensions and both shipped layouts).
2. **Root group**: one `<g>` containing the background and all buttons.
3. **Background**: a full-canvas shape as the first child — CSS paints it with `--color-bg` at `--bg-opacity`.
4. **Button groups**: each button is a `<g id="BUTTON_ID">`. The runtime splits each ring path into a FILL (solid outer subpath, invisible in idle, full-color when pressed) + OUTLINE (ring, always visible). Groups ending in `_reach` are boundary rings (outline-only, 100% opacity).
5. **Layer IDs**: must match the naming conventions below — JS maps them to Standard Gamepad API button indices.

### Naming conventions

| ID | Gamepad button index | Element |
| -- | -------------------- | ------- |
| `a`, `b`, `x`, `y` | 0, 1, 2, 3 | Face buttons |
| `lb`, `rb` | 4, 5 | Bumpers (shoulder buttons) |
| `lt`, `rt` | 6, 7 | Triggers (analog) |
| `view`, `menu` | 8, 9 | View/Menu (or Share/Options, or −/+) |
| `l3`, `r3` | 10, 11 | Stick-click buttons (arcade: separate round buttons) |
| `up`, `down`, `left`, `right` | 12–15 | D-pad (Xbox layout) |
| `left_joy`, `right_joy` | — | Analog stick knobs (Xbox; double as l3/r3) |
| `left_joy_reach`, `right_joy_reach` | — | Analog stick boundary rings |
| `joy`, `joy_reach` | — | Arcade stick + boundary |

Notes:

- The `_reach` suffix is special — those elements render as outline-only rings.
- The arcade layout has no D-pad groups; the stick captures D-pad input (see [Stick trail](#arcade-stick-trail)).
- In the Xbox layout, L3/R3 are the stick knobs themselves (pressing LS/RS).

### Workflow

1. **Draw** the controller in Inkscape (or Affinity Designer): 640×216 document, one layer per button named with the button ID (lowercase), 2-path structure per button (outer shape + inner shape, ~2px gap between them — the "stroke" width).
2. **Export as Plain SVG** (Inkscape: File → Save As → Plain SVG; Affinity: export SVG with IDs preserved). Verify the exported file has `<g id="...">` elements matching your layer names.
3. **Place** the SVG at `masks/controllers/<layout>.svg`.
4. **Inline** it into `index.html`: strip the `<?xml>` declaration and `<!DOCTYPE>`, and paste the `<svg>…</svg>` into a `<template id="svg-<layout>">` element.
5. **Register** the controller in the `CONTROLLERS` map in the JS:

   ```js
   const CONTROLLERS = Object.freeze({
       xbox: 'svg-xbox',
       arcade: 'svg-arcade',
       your_layout: 'svg-your_layout'  // new
   });
   ```

6. **Test** with `?controller=your_layout` in a browser and in OBS (see [Testing in OBS](#testing-in-obs)).

**Anti-patterns:**

- Don't hardcode button positions in CSS — the SVG is the source of truth.
- Don't use `fetch()` to load SVGs at runtime — OBS `file://` blocks it. Always inline.
- Don't include brand-protected art (Sony/Microsoft/Nintendo logos or trademarked button icons) — use generic shapes; users can add their own branded art locally.

### Label sprite packs (advanced)

For distributing complete icon sets as a single file, the project ships a **sprite sheet model** at `masks/labels/sprite_example.svg`: one SVG containing multiple `<symbol>` elements, each referenced by ID via `<use>`:

```xml
<!-- In the sprite file -->
<symbol id="icon-a" viewBox="0 0 32 32">
    <path d="..." stroke="currentColor" fill="none" />
</symbol>

<!-- In the overlay (inside each button group) -->
<svg class="btn-icon" viewBox="0 0 32 32">
    <use href="masks/labels/sprite.svg#icon-a" />
</svg>
```

```css
.btn-icon {
    position: absolute; top: 50%; left: 50%;
    width: 70%; height: 70%;
    transform: translate(-50%, -50%);
    fill: var(--button-color, var(--accent));
    pointer-events: none;
}
```

Key rules for your own packs:

- Use `fill="currentColor"` (or `stroke="currentColor"`) inside every `<symbol>` — the icon inherits the button's `--color-*` variable this way. A hardcoded fill breaks per-button customization.
- Use one `<symbol id="icon-<button>">` per button (`icon-a`, `icon-b`, … `icon-view`, `icon-menu`), all with the same viewBox (recommended `0 0 32 32`).
- Use pure SVG elements (`<path>`, `<text>`, `<circle>`, `<rect>`) — never `<image>` with external references.
- Test: change `--color-a: #FF0000` in DevTools — the A icon should turn red. If not, a fixed fill slipped into a symbol.

When to use what: for 4 simple text labels, the `--label-*` variables (or `?icons=`) are enough — no sprite needed. Sprite packs pay off with 10+ complex icons or distributable theme packs. The runtime integration of external sprite files is a roadmap item; the model file and this recipe are ready for anyone who wants to wire it today.

---

## Testing in OBS

The overlay working in a browser ≠ working in OBS. OBS's CEF (Chromium Embedded Framework) has quirks. Test in OBS before considering a change done.

**Minimum test matrix:**

1. **No controller connected** — overlay renders centered, no console errors, hint appears after 5s.
2. **With a controller** — press each button; the corresponding element lights up with the pressed state.
3. **Analog sticks** — move to extremes; knobs stay within the boundary rings (no overflow).
4. **Triggers** — press LT/RT partially; the fill grows proportionally (pressure-sensitive).
5. **URL params** — test `?scale=2`, `?fit=cover`, `?icons=playstation`, `?color=xbox` (separately and combined).
6. **Arcade layout** — `?controller=arcade`: stick moves, trail follows in Nox→Sol gradient.
7. **Debug mode** — double-click the overlay; guide borders and the hint appear.
8. **Reload** — right-click the source → Refresh; the overlay reloads cleanly.
9. **Multiple source sizes** — `640×216` (base), `1920×648` (full-bleed), `1920×1080` (16:9 with padding).

**Console access in OBS:**

1. Right-click the Browser Source → **Interact**.
2. Right-click inside the interaction window → **Inspect Element**.
3. **Console** tab — errors and logs (including the `?gamepad=list` output).

**Common CEF-specific issues:**

- `color-mix()` not supported → upgrade OBS (CEF 111+ required).
- Gamepad not detected in OBS but works in browser → check OS-level gamepad permissions; on Windows, verify the controller in the Game Controllers control panel.

---

## Contributing

Public-domain project — there's no formal contribution process. Fork, modify, send a PR if you want. The guidelines below make life easier for everyone.

### Reporting bugs

Open an issue with: OBS version, OS, controller (model + USB/Bluetooth), the Browser Source URL (including `?` params), expected vs. actual behavior, a screenshot/video if relevant, and console errors (see [Console access](#testing-in-obs)).

### Code conventions

- **Single-file**: everything lives in `index.html`. No external CSS/JS files.
- **CSS variables for everything** a user might want to customize (colors, sizes, timing, opacity) — this is the customization API.
- **`var()` with fallback**: `color: var(--button-color, var(--accent))` so missing variables don't break the layout.
- **`color-mix()` is OK** (CEF 111+). Avoid `@layer` (flaky in some CEF versions).
- **Vanilla JS only**: no bundlers, no transpilers, no `npm install`. IIFE wrapper, class-based architecture, `requestAnimationFrame` for the polling loop, `URLSearchParams` for params.
- **Naming**: lowercase-hyphen filenames, PascalCase CSS classes, camelCase JS.
- **Comments in English** for new code. **Never** put `--` (double hyphen) inside XML comments — XML forbids it.

### What NOT to contribute

- **WebHID** — removed for stability reasons; the Standard Gamepad API covers all realistic OBS overlay needs.
- **Build pipelines** — the single-file HTML constraint is intentional. If you need a build, do it as a separate fork.
- **External dependencies** — no npm packages, no CDNs, no fonts from Google Fonts. The overlay must run offline, from `file://`.
- **Brand-protected art** — no Sony/Microsoft/Nintendo logos or trademarked icons. Use generic shapes.

### License

By contributing, you agree that your contributions are licensed under [The Unlicense](LICENSE) (public domain). No attribution will be required for your contributions.

---

## Architecture invariants

These are the project's hard constraints. A change that violates any of them is a defect, even if it "works":

1. **Single-file HTML** — `index.html` is the only runtime file. All HTML, CSS, and JS are inline. No `<link>`, no `<script src=>` to local files. Must run from `file://` without a server.
2. **No WebHID** — Standard Gamepad API only.
3. **No build pipelines, no external dependencies** — no npm, bundlers, transpilers, CDNs, or external fonts. Dev tooling is fine as long as it's never required to *run* the overlay; a build step that *produces* `index.html` is forbidden.
4. **No brand-protected art** — generic shapes only; users add their own branded art locally.
5. **SVG blueprints inline** — all layouts are `<template>` SVGs cloned at runtime, with positions coming from the SVG itself (not CSS-positioned HTML).
6. **Offline-first / `file://` compatible** — no `fetch()` of local files, no ES modules with relative imports, no service workers. Vanilla JS in an IIFE is the contract.
7. **CSS variables as the customization API** — everything a user might tune is a `:root` variable. URL params (`?color=`, `?icons=`, …) are the live API; `:root` variables are the static API. Both stay in sync and documented.
8. **CEF 111+ compatibility** — verify any new CSS/JS feature against OBS's embedded Chromium before merging.

---

## Documentation

| File | What it covers |
| ---- | -------------- |
| [README.md](README.md) | This file — usage, full customization reference, layout creation, contributing |
| [CHANGELOG.md](CHANGELOG.md) | Versioned technical history |
| [ROADMAP.md](ROADMAP.md) | Status, planned variants, future technical improvements |
| [AGENT.md](AGENT.md) | Operating profile for AI-assisted maintenance (universal, identical across the maintainer's projects) |

The overlay at runtime only needs `index.html` (SVGs are inlined). The `masks/controllers/` SVGs are the source-of-truth blueprints — edit them, then re-inline into `index.html`.

## Project structure

```
.
├── index.html                  # The overlay (single file — HTML + CSS + JS + inline SVGs)
├── masks/
│   ├── controllers/
│   │   ├── xbox.svg            # Xbox blueprint (source of truth, 640×216)
│   │   └── arcade.svg          # Arcade fightstick blueprint (source of truth, 640×216)
│   └── labels/
│       └── sprite_example.svg  # Label sprite pack model (see Label sprite packs)
├── README.md                   # This file
├── CHANGELOG.md                # Versioned technical history
├── ROADMAP.md                  # Public roadmap
├── AGENT.md                    # Universal agent operating profile
├── VERSION                     # Current version (v1.YYMM.XXX)
└── LICENSE                     # The Unlicense (public domain)
```

## Roadmap (high-level)

The Xbox and Arcade layouts are the **open core** of the project. Variants under consideration: PlayStation DualSense (✕ ○ □ △, touchpad) and Nintendo Pro Controller (swapped A/B/X/Y). See [ROADMAP.md](ROADMAP.md).

---

## License

**[The Unlicense](LICENSE)** — public domain dedication. You can use, copy, modify, publish, distribute, sublicense, and sell this software without any restriction. No attribution required.
