# `masks/` — Overlay assets

This folder contains the **source-of-truth assets** that define the overlay's visual identity. As of **Track C** (the current architecture, see [VARIANTS.md](../VARIANTS.md)), the overlay is drawn entirely from inline SVG — there is no longer a PNG background image.

---

## What's here

```
masks/
├── controllers/   # SVG blueprints for each controller layout (source of truth)
│   ├── xbox.svg
│   ├── arcade.svg
│   └── README.md  # How to create/edit controller SVGs
└── labels/        # SVG sprite-sheet model for icon packs (Phase 5, planned)
    ├── sprite_example.svg
    └── README.md  # How to create sprite packs
```

> **Historical note (Track A/B → Track C):** Earlier versions of this project shipped a `masks/bg_dark.png` (3840×2160) file used as the overlay's visual background, with CSS cropping a 768×324 region of it. That architecture was **superseded by Track C**: the controller art now lives in `masks/controllers/*.svg` (viewBox 768×324), inlined as `<template>` elements in `index.html` and animated by JS. The PNG background was removed in `v1.2610.001`. See [CHANGELOG.md](../CHANGELOG.md) and [docs/AUDIT.md](../docs/AUDIT.md) for the migration history.

---

## How the overlay renders (Track C)

1. The SVG in `masks/controllers/<layout>.svg` is the **source of truth** for that layout's button positions, sizes, and shapes.
2. The maintainer inlines the SVG into `index.html` as `<template id="svg-<layout>">` (so the overlay works from `file://` in OBS without runtime `fetch()`).
3. At runtime, JS clones the appropriate template based on `?controller=`, strips inline styles, and animates each `<g id="<button>">` based on Gamepad API input.
4. CSS controls all fills, opacities, and pressed states via `--color-*` variables in `:root`.

To add a new controller layout → see [controllers/README.md](controllers/README.md).
To create an icon sprite pack → see [labels/README.md](labels/README.md).

---

## Backgrounds and theming (Track C)

Track C does **not** use a background image. The overlay's background is a CSS-drawn rect (the first child of each SVG's root `<g id="Prancheta1">`), whose fill is controlled by `--color-bg` and `--bg-opacity` in `:root`.

- **Dark theme (default):** `--accent: #FFFFFF` over `--color-bg: #000000` → white controller on black background.
- **Light theme:** swap `--accent: #000000` over `--color-bg: #FFFFFF` → dark controller on white background.
- **Auto light/dark:** optional `@media (prefers-color-scheme)` block (see [CUSTOMIZATION_GUIDE.md](../CUSTOMIZATION_GUIDE.md)).

If you want a **custom branded background** behind the controller (e.g., a streamer's channel art), add it as a separate OBS source layer behind the Bubo Browser Source — the overlay itself stays a single-file HTML with a CSS-controlled background, preserving the offline-first, no-external-requests invariant.

---

## Creating custom assets

| Asset type | Guide | Notes |
| --- | --- | --- |
| New controller layout (DualSense, Nintendo Pro, custom) | [controllers/README.md](controllers/README.md) | SVG, viewBox 768×324, named `<g id="...">` per button |
| Icon sprite pack (custom labels) | [labels/README.md](labels/README.md) | SVG sprite sheet, Phase 5 (model ready, runtime integration planned) |
| Custom colors / opacity / stick trail | [CUSTOMIZATION_GUIDE.md](../CUSTOMIZATION_GUIDE.md) | CSS variables in `:root` |
| Grid, proportions, z-index, animation timing | [DESIGN_GUIDE.md](../DESIGN_GUIDE.md) | Visual doctrine for variants |

### Common mistakes (legacy)

The following issues are **legacy** from the PNG-based Track A/B architecture and no longer apply in Track C. They are preserved here as a migration reference for anyone forking from an older version.

- **Buttons misaligned with the image art** — Track C: alignment comes from the SVG, not from a base image. If buttons are misaligned, edit the SVG in `masks/controllers/`.
- **Image is pixelated** — Track C: there is no PNG. The SVG scales crisply at any resolution.
- **Image doesn't appear** — Track C: there is no image path. If the controller doesn't render, check that `index.html` contains the `<template id="svg-<layout>">` and the `CONTROLLERS` map includes the layout.
- **Solid colored background bleeding around the controller** — Track C: the SVG's background rect is exactly 768×324, matching the overlay stage. There is no bleed. If you see bleed, check `--bg-opacity` and the SVG's `<rect x="0" y="0" width="768" height="324"/>`.

---

## See also

- [VARIANTS.md](../VARIANTS.md) — the 3-track evolution (A → B → C) and the SVG workflow.
- [DESIGN_GUIDE.md](../DESIGN_GUIDE.md) — grid, proportions, z-index, animation timing (Track C context).
- [CUSTOMIZATION_GUIDE.md](../CUSTOMIZATION_GUIDE.md) — colors, opacity, stick trail, label sets.
- [masks/controllers/README.md](controllers/README.md) — SVG blueprint authoring guide.
- [masks/labels/README.md](labels/README.md) — sprite pack model (Phase 5).
- [docs/AUDIT.md](../docs/AUDIT.md) — internal audit history (PT-BR), including the Track A → Track C migration.
