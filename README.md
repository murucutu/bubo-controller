# Bubo Controller

**An owl-eyed gamepad overlay for OBS Browser Source.**

[![License: Unlicense](https://img.shields.io/badge/license-Unlicense-blue.svg)](LICENSE)
[![Made for OBS](https://img.shields.io/badge/made%20for-OBS-9146FF.svg)](https://obsproject.com/)
[![Language: PT-BR](https://img.shields.io/badge/lang-EN%20%7C%20PT--BR-success.svg)](#languages)
![Version](https://img.shields.io/badge/version-v1.2610.003-9146FF.svg)

**Version:** `v1.2610.003` · **Scheme:** `v1.YYMM.XXX` (v1 = published · YYMM = build month · XXX = sprint number, resets monthly) — see [CHANGELOG.md](CHANGELOG.md).

**Languages:** English · [Português (BR)](README.pt.md)

---

## About

Bubo Controller is a single-file HTML overlay that reads the browser's [Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API) and draws a gamepad on your OBS scene in real time. It supports multiple controller layouts (Xbox, Arcade fightstick) via swappable SVG blueprints. Triggers are pressure-sensitive, analog sticks are calibrated, and every button color is individually customizable.

**Why "Bubo"?** *Bubo* is the Latin name for the horned owl and the owl that accompanied Minerva, the Roman goddess of wisdom. The owl has watched over streamers since ancient times — and now it watches your gamepad. 🦉

The project is dedicated to the **public domain** ([The Unlicense](LICENSE)) — use it, fork it, sell it, derive from it. No attribution required.

---

## Features

- ✅ Single-file HTML — no build, no dependencies, no external requests
- ✅ **Multiple controller layouts** via SVG blueprints (`?controller=xbox|arcade`)
- ✅ Standard Gamepad API (works with Xbox, PlayStation, generic controllers)
- ✅ Pressure-sensitive triggers LT/RT with reactive waveform graph (Xbox layout)
- ✅ Calibrated analog sticks with dead zone
- ✅ **Arcade layout** with ghost-glow stick trail (1/3 knob size, analog + D-pad capture)
- ✅ Per-button colors via CSS variables (`--color-a`, `--color-b`, …)
- ✅ **ABXY color schemes** via URL (`?color=xbox|playstation`)
- ✅ Button labels swappable via URL (`?icons=xbox|playstation|nintendo|none`)
- ✅ **Opacity layering system** — 40% background + 40% buttons = ~64% combined; 100% on press
- ✅ Auto-scales to any OBS Browser Source size while preserving aspect ratio
- ✅ Optional fit modes (`?fit=contain|cover`)
- ✅ Debug mode (double-click the overlay) for visual inspection
- ✅ SVG-based art (Track C architecture — inline SVG, JS-animated)

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
| `?controller=` | `xbox` (default) / `arcade` | Controller layout to display |
| `?color=` | `xbox` / `playstation` | ABXY color scheme (default: all white) |
| `?icons=` | `xbox` (default) / `playstation` / `nintendo` / `none` | Button label text set |
| `?scale=X` | any positive number | Manual scale override (e.g. `?scale=2` = 2× zoom) |
| `?fit=` | `contain` (default) / `cover` | How overlay fits the source: contain = preserve aspect with padding; cover = fill source, may clip sides |

Combine freely: `?controller=arcade&color=xbox&icons=playstation&scale=2`

---

## Customization

All configuration lives in `:root` at the top of the `<style>` block in `index.html`. No build step.

```css
:root {
    --accent: #FFFFFF;        /* global color (white on black, or swap for dark on white) */
    --color-bg: #000000;      /* background tint (used for label text contrast) */

    /* ABXY colors (set by ?color= or manually) */
    --color-a: var(--accent);
    --color-b: var(--accent);
    --color-x: var(--accent);
    --color-y: var(--accent);

    /* Opacity system */
    --bg-opacity: 0.4;              /* background rect fill opacity */
    --button-opacity: 0.4;          /* unpressed button fill opacity */
    --button-pressed-opacity: 1;    /* pressed button fill opacity */

    /* Stick trail (Arcade only) */
    --stick-trail-length: 30;       /* number of trail frames */
    --stick-trail-fade: 0.04;       /* fade rate per frame (lower = softer) */
    --stick-trail-radius: 12;       /* trail circle radius (1/3 of knob) */
    --stick-trail-blur: 12px;       /* ghost glow blur */
}
```

See **[CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md)** for full palettes, opacity tuning, stick trail tuning, and how to create custom controller SVGs.

---

## Documentation

| File | Audience | What it covers |
| ---- | -------- | --------------- |
| [AGENT.md](AGENT.md) | Agent / Senior PM | Operating manual for the AI agent acting as Senior PM (versioning policy, invariants, never-rules, DoD) |
| [CHANGELOG.md](CHANGELOG.md) | Everyone | Versioned history, with verbatim conversation excerpts per Sprint |
| [CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md) | Users | Colors, opacity system, stick trail tuning, label sets |
| [DESIGN_GUIDE.md](DESIGN_GUIDE.md) | Designers | Grid, proportions, spacing, z-index, animation timing |
| [VARIANTS.md](VARIANTS.md) | Designers/Contributors | SVG blueprint workflow; how to add new controller layouts |
| [LABEL_PACKS_GUIDE.md](LABEL_PACKS_GUIDE.md) | Advanced | SVG sprite sheets for icon packs |
| [ROADMAP.md](ROADMAP.md) | Contributors | Status, planned variants, contribution hooks |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contributors | How to fork, build, test, and submit PRs |
| [masks/controllers/README.md](masks/controllers/README.md) | Artists | How to create/edit controller SVG blueprints |
| [masks/README.md](masks/README.md) | Artists | How to create custom assets |
| [masks/labels/README.md](masks/labels/README.md) | Advanced | How to create sprite packs for label icons |
| [docs/AUDIT.md](docs/AUDIT.md) | Maintainers | Internal design audit history (in Portuguese) |

The overlay at runtime only needs `index.html` (SVGs are inlined). The `masks/controllers/` SVGs are the source-of-truth blueprints — edit them, then re-inline into `index.html`.

---

## Project Structure

```
.
├── index.html                  # Unified overlay (Xbox + Arcade, SVG inline)
├── masks/
│   ├── controllers/
│   │   ├── xbox.svg            # Xbox controller SVG blueprint (source of truth)
│   │   ├── arcade.svg          # Arcade fightstick SVG blueprint (source of truth)
│   │   └── README.md           # How to create/edit controller SVGs
│   ├── README.md               # Custom assets guide
│   └── labels/
│       ├── sprite_example.svg  # SVG sprite pack model (Phase 5)
│       └── README.md           # How to create sprite packs
├── AGENT.md                    # Senior PM operating manual (versioning, invariants, DoD)
├── CHANGELOG.md                # Versioned history (with conversation excerpts)
├── VERSION                     # Current version (v1.YYMM.XXX)
├── CUSTOMIZATION_GUIDE.md      # Colors, opacity, stick trail, labels
├── DESIGN_GUIDE.md             # Grid, proportions, timing
├── VARIANTS.md                 # SVG workflow for new controller layouts
├── LABEL_PACKS_GUIDE.md        # SVG sprite sheets
├── ROADMAP.md                  # Public roadmap
├── CONTRIBUTING.md             # How to contribute
├── README.md                   # This file (English)
├── README.pt.md                # Portuguese version
├── LICENSE                     # The Unlicense (public domain)
├── .gitignore
└── docs/
    ├── AUDIT.md                # Internal audit history (PT-BR)
    └── AUDIT_PLAN.md           # Audit planning notes (PT-BR)
```

---

## Roadmap (public, high-level)

The Xbox and Arcade layouts are the **open core** of the project. Variants for other controllers are planned for the future. Detailed plans are kept private by the maintainer during development.

Variants under consideration:
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

- **Etymology**: *Bubo* — Latin for horned owl. In Roman mythology, Bubo was the owl familiar of Minerva, goddess of wisdom.
- **Etymology (PT-BR)**: The maintainer's nickname "Murucutu" comes from Tupi-Guarani for owl (*Asio clamator*).
- **Standard Gamepad API** — for being stable enough to drop WebHID entirely.
- **OBS Studio** — for being excellent software and free.

---

## License

**[The Unlicense](LICENSE)** — public domain dedication. You can use, copy, modify, publish, distribute, sublicense, and sell this software without any restriction. No attribution required.
