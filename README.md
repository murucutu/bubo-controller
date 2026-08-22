# Bubo Controller

**An owl-eyed gamepad overlay for OBS Browser Source.**

[![License: Unlicense](https://img.shields.io/badge/license-Unlicense-blue.svg)](LICENSE)
[![Made for OBS](https://img.shields.io/badge/made%20for-OBS-9146FF.svg)](https://obsproject.com/)
[![Language: PT-BR](https://img.shields.io/badge/lang-EN%20%7C%20PT--BR-success.svg)](#languages)

**Languages:** English · [Português (BR)](README.pt.md)

---

## About

Bubo Controller is a single-file HTML overlay that reads the browser's [Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API) and draws an Xbox-style controller on your OBS scene in real time. Triggers are pressure-sensitive, analog sticks are calibrated, and every button color is individually customizable.

**Why "Bubo"?** *Bubo* is the Latin name for the horned owl and the owl that accompanied Minerva, the Roman goddess of wisdom. The owl has watched over streamers since ancient times — and now it watches your gamepad. 🦉

The project is dedicated to the **public domain** ([The Unlicense](LICENSE)) — use it, fork it, sell it, derive from it. No attribution required.

---

## Features

- ✅ Single-file HTML — no build, no dependencies, no external requests (except `masks/bg_dark.png`)
- ✅ Standard Gamepad API (works with Xbox, PlayStation, generic controllers)
- ✅ Pressure-sensitive triggers LT/RT with reactive waveform graph
- ✅ Calibrated analog sticks LS/RS with dead zone
- ✅ Per-button colors via CSS variables (`--color-a`, `--color-b`, …)
- ✅ Button labels swappable via URL (`?icons=xbox|playstation|nintendo|none`)
- ✅ Auto-scales to any OBS Browser Source size while preserving aspect ratio
- ✅ Optional fit modes (`?fit=contain|cover`)
- ✅ Debug mode (double-click the overlay) for visual inspection
- ✅ Accessibility: `aria-label` on interactive elements, `aria-hidden` on decorative

---

## Quick Start

1. **Download & unzip** the project (or `git clone https://github.com/murucutu/bubo-controller.git`).
2. In OBS Studio, add a **Browser** source.
3. Check **Local file** and select `index.html`.
4. Set **Width** and **Height**. For full-bleed (no transparent padding), use any ~2.37:1 size:
   - `768 × 324` (base, scale 1×)
   - `1920 × 810` (2.5× — HD full-bleed horizontal)
   - `3840 × 1620` (5× — 4K full-bleed horizontal)
   - Or any 16:9 (`1920×1080`, `3840×2160`) — overlay fills width, with vertical padding.
5. Plug in a gamepad. Press any button. The overlay reacts in real time.

> Tip: enable **"Shutdown source when not visible"** to save resources when the scene is inactive.

---

## URL Parameters

| Parameter | Values | Description |
| --------- | ------ | ----------- |
| `?scale=X` | any positive number | Manual scale override (e.g. `?scale=2` = 2× zoom) |
| `?fit=` | `contain` (default) / `cover` | How overlay fits the source: contain = preserve aspect with padding; cover = fill source, may clip sides |
| `?icons=` | `xbox` (default) / `playstation` / `nintendo` / `none` | Button label set |

Combine freely: `?fit=cover&icons=playstation&scale=2`

---

## Customization

All configuration lives in `:root` at the top of the `<style>` block in `index.html`. No build step.

```css
:root {
    --accent: #FFFFFF;        /* fallback color for any button */
    --color-bg: #000000;      /* background tint reference */

    /* Per-button colors (default = --accent) */
    --color-a: var(--accent);
    --color-b: var(--accent);
    --color-x: var(--accent);
    --color-y: var(--accent);
    /* ...etc */

    --label-a: "A";          /* text label per button */
    --label-b: "B";
    /* ...etc */

    --stroke: 5px;            /* border thickness */
    --opacity: 0.80;          /* overall overlay opacity */
}
```

Make the A button red:

```css
:root { --color-a: #FF0000; }
```

See **[CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md)** for full palettes, button shapes, custom backgrounds, and label sets.

---

## Documentation

| File | Audience | What it covers |
| ---- | -------- | --------------- |
| [CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md) | Users | Colors, button shapes, labels, custom backgrounds |
| [DESIGN_GUIDE.md](DESIGN_GUIDE.md) | Designers | Grid, proportions, spacing, z-index, animation timing |
| [LABEL_PACKS_GUIDE.md](LABEL_PACKS_GUIDE.md) | Advanced | SVG sprite sheets (`<symbol>` + `<use>`) for icon packs |
| [ROADMAP.md](ROADMAP.md) | Contributors | Status, planned variants, contribution hooks |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contributors | How to fork, build, test, and submit PRs |
| [masks/README.md](masks/README.md) | Artists | How to create custom `bg_dark.png` from scratch |
| [masks/labels/README.md](masks/labels/README.md) | Advanced | How to create sprite packs for label icons |
| [docs/AUDIT.md](docs/AUDIT.md) | Maintainers | Internal design audit history (in Portuguese) |

The overlay at runtime only needs `index.html` + `masks/bg_dark.png`. Everything else is documentation.

---

## Project Structure

```
.
├── index.html              # Overlay (HTML + CSS + JS inline, single file)
├── masks/
│   ├── bg_dark.png         # Background art (3840×2160)
│   ├── README.md           # How to create custom backgrounds
│   └── labels/
│       ├── sprite_example.svg  # SVG sprite pack model (Phase 5)
│       └── README.md           # How to create sprite packs
├── CUSTOMIZATION_GUIDE.md  # Colors, shapes, labels
├── DESIGN_GUIDE.md         # Grid, proportions, timing
├── LABEL_PACKS_GUIDE.md    # SVG sprite sheets
├── ROADMAP.md              # Public roadmap
├── CONTRIBUTING.md         # How to contribute
├── README.md               # This file (English)
├── README.pt.md            # Portuguese version
├── LICENSE                 # The Unlicense (public domain)
├── .gitignore
└── docs/
    ├── AUDIT.md            # Internal audit history (PT-BR)
    └── AUDIT_PLAN.md       # Audit planning notes (PT-BR)
```

---

## Roadmap (public, high-level)

The base Xbox layout is the **open core** of the project. Variants for other controllers are planned for the future. Detailed plans (layouts, button mappings, architecture decisions) are kept private by the maintainer during development.

Variants under consideration:
- Arcade fightstick for fighting games
- PlayStation DualSense with ✕ ○ □ △ symbols and touchpad
- Nintendo Pro Controller with swapped A/B/X/Y layout

See [ROADMAP.md](ROADMAP.md) for the public high-level roadmap.

---

## Contributing

This is a public-domain project. There's no formal contribution process — fork it, modify it, send a PR if you want. See [CONTRIBUTING.md](CONTRIBUTING.md) for code conventions and OBS testing tips.

### What NOT to contribute

- WebHID (removed — see [docs/AUDIT.md](docs/AUDIT.md))
- Build pipelines (the single-file HTML constraint is intentional)
- External dependencies (the overlay must work offline)

---

## Acknowledgments

- **Etymology**: *Bubo* — Latin for horned owl. In Roman mythology, Bubo was the owl familiar of Minerva, goddess of wisdom. The owl's reputation for night-vision and vigilance fits an overlay that watches your controller.
- **Etymology (PT-BR)**: The maintainer's nickname "Murucutu" comes from Tupi-Guarani for owl (*Asio clamator* — striped owl), tying the Latin and Tupi traditions together.
- **Standard Gamepad API** — for being stable enough to drop WebHID entirely.
- **OBS Studio** — for being excellent software and free.

---

## License

**[The Unlicense](LICENSE)** — public domain dedication. You can use, copy, modify, publish, distribute, sublicense, and sell this software without any restriction. No attribution required.

> The maintainer offers **paid customization services** (custom-themed overlays for streamers). The open-source base remains free forever. If you want a custom variant (themed, branded, or proprietary layout), contact the maintainer via GitHub or your usual streaming channels.
