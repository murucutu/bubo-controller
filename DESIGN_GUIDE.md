# Design Guide

This guide documents the overlay's proportion system, spacing, and layers, so anyone can:

- understand why each dimension exists,
- plan a variant (arcade fightstick, DualSense, Nintendo Pro Controller, etc.),
- adjust positions without breaking visual balance.

Everything here is reference — the actual source of truth is in `index.html`. This guide is the visual doctrine of the project.

> **Track C note:** This guide documents the Xbox layout's proportions, spacing, and layers. In the current architecture (Track C, see [VARIANTS.md](VARIANTS.md)), the controller art lives in `masks/controllers/xbox.svg` (viewBox `0 0 768 324`) and is inlined into `index.html` as a `<template>`. The dimensions below are the **reference values** encoded in that SVG; if you edit the SVG, you are editing the source of truth. The earlier Track A/B architecture used a 3840×2160 `masks/bg_dark.png` base image with a CSS crop — that approach is **superseded** and the PNG has been removed (see [docs/AUDIT.md](docs/AUDIT.md) and [CHANGELOG.md](CHANGELOG.md) for the migration history).

---

## Table of Contents

- [Canvas system (SVG viewBox)](#canvas-system-svg-viewbox)
- [Overlay grid (controller's natural aspect)](#overlay-grid-controllers-natural-aspect)
- [Controller drawing proportions](#controller-drawing-proportions)
- [Button table (dimensions and positions)](#button-table-dimensions-and-positions)
- [Spacing and visual rhythm](#spacing-and-visual-rhythm)
- [Z-index hierarchy](#z-index-hierarchy)
- [Animation timing](#animation-timing)
- [Planning a variant](#planning-a-variant)

---

## Canvas system (SVG viewBox)

All controller art is drawn on an SVG canvas of **768×324** (`viewBox="0 0 768 324"`). This single canvas is both the visual overlay and the master proportions reference — everything in the SVG (and everything derived in CSS/JS) is positioned within these 768×324 units.

> **Legacy (Track A/B):** Earlier versions used a 3840×2160 base image (`masks/bg_dark.png`) and CSS-cropped a 768×324 region from it. Track C replaced this with a direct 768×324 SVG canvas — no base image, no crop. The 3840×2160 image no longer ships with the project.

### Canonical divisions (in the 768×324 canvas)

| Region                   | Width (768)        | Height (324)       | Pixels         |
| ------------------------ | ------------------ | ------------------ | -------------- |
| Controller width         | 100% (768px)       | —                  | 768px          |
| Controller height        | —                  | 100% (324px)       | 324px          |
| Top row (LT/RT)          | —                  | 33.3%              | 108px          |
| Bottom row (rest)        | —                  | 66.7%              | 216px          |

### Overlay stage = controller canvas

The visible overlay region **is** the entire 768×324 SVG canvas. There is no crop, no padding — the SVG fills the overlay stage exactly:

```
0,0 ─────────────────────── 768,0
  │                          │
  │   768 × 324 (overlay =    │
  │       controller SVG)    │
  │                          │
0,324 ───────────────────── 768,324
```

> If you create a variant with a different controller size (e.g., a taller arcade fightstick canvas), you change the SVG's viewBox. The overlay stage in `index.html` is driven by `--overlay-width` / `--overlay-height` CSS variables; keep them in sync with the SVG viewBox.

---

## Overlay grid (controller's natural aspect)

The official overlay is **768×324 (~2.37:1)** — exactly the controller drawing size. **Zero transparent padding** inside the overlay.

```
+--------------------------------+
|                                |
|   768 × 324 (overlay = controller)|
|                                |
+--------------------------------+
```

> History: an earlier version used a 1280×720 (16:9) overlay with the controller centered, but this created ~256px of horizontal and ~198px of vertical transparent padding. On 16:9 sources (1920×1080, 3840×2160), the controller appeared small inside a large transparent frame. We reverted to the controller's natural aspect (768×324) to eliminate internal padding. (In Track A/B, this also matched the `bg_dark.png` crop region; in Track C, it matches the SVG viewBox directly.)

### Recommended aspect ratios for OBS Browser Source

For **no transparent padding** (full-bleed), use a source with ~2.37:1 aspect:

| Source size        | Aspect          | Scale | Typical use                              |
| ------------------ | --------------- | ----- | ---------------------------------------- |
| 768 × 324          | 2.37:1 (base)   | 1×    | Compact layout, mini-views              |
| 1536 × 648         | 2.37:1          | 2×    | HD horizontal full-bleed                 |
| 1920 × 810         | 2.37:1          | 2.5×  | Full HD horizontal full-bleed           |
| 3840 × 1620        | 2.37:1          | 5×    | 4K horizontal full-bleed                 |

For **16:9** sources (1920×1080, 3840×2160) with `?fit=contain` (default), the overlay fills the width and leaves vertical transparent padding (~270px on a 4K source). To fill 100%, use `?fit=cover` (warning: clips LT/RT).

| Source size        | Mode             | Scale | Result                                                |
| ------------------ | ---------------- | ----- | ----------------------------------------------------- |
| 1920 × 1080        | ?fit=contain     | 2.5×  | 1920×810 + 135px padding top/bottom                   |
| 1920 × 1080        | ?fit=cover       | 3.33× | 2560×1080 visual, 320px clipped on each side (LT/RT)  |
| 3840 × 2160        | ?fit=contain     | 5×    | 3840×1620 + 270px padding top/bottom                  |
| 3840 × 2160        | ?fit=cover       | 6.67× | 5120×2160 visual, 640px clipped on each side          |

> Recommendation: if you control the source size, prefer 2.37:1 (1920×810 or 3840×1620). Only use `?fit=cover` on a 16:9 source if you need the controller "stretched" and accept losing the LT/RT triggers.

---

## Controller drawing proportions

Within the 768×324 crop, the controller has two horizontal rows:

| Row                  | Height | Use                            |
| -------------------- | ------ | ------------------------------ |
| Top row              | 108px  | LT, RT, LB, RB, View, Menu    |
| Bottom row           | 216px  | D-pad, ABXY, analog sticks LS/RS |

```
+----[ 768 × 324 controller drawing ]----+
| 108px: LT LB View Menu RB RT          |
|----------------------------------------|
| 216px: D-pad    ABXY    LS      RS     |
+----------------------------------------+
```

### Golden ratio and zones

The top row occupies 1/3 of the controller height (108/324 = 33.3%), the bottom row occupies 2/3 (216/324 = 66.7%). This follows the rule of thirds for visual balance.

Horizontally, the controller has 3 equidistant functional zones:

| Zone          | X range (in controller) | Main content              |
| ------------- | ----------------------- | ------------------------- |
| Left          | 0–256px                 | LT/LB, LS analog stick    |
| Center        | 256–512px               | View, Menu, D-pad          |
| Right         | 512–768px               | RB/RT, ABXY, RS analog stick|

Each zone has ~256px = 1/3 of the controller. Center elements (D-pad, View/Menu) are aligned in the middle of the center zone.

---

## Button table (dimensions and positions)

All positioning is within the 768×324 controller-stage. Coordinates in pixels at base scale.

### Top row (y = 0–108)

| Button | left    | top  | width | height | border-radius | z-index |
| ------ | ------- | ---- | ----- | ------ | -------------- | ------- |
| LT     | 10px    | 23px | 185px | 62px   | 12px           | 4       |
| LB     | 210px   | 23px | 92px  | 62px   | 12px           | 3       |
| View   | 326px   | 25px | 58px  | 58px   | 999px 0 0 999px| 3       |
| Menu   | 384px   | 25px | 58px  | 58px   | 0 999px 999px 0| 3       |
| RB     | 464px   | 23px | 92px  | 62px   | 12px           | 3       |
| RT     | 573px   | 23px | 185px | 62px   | 12px           | 4       |

### Bottom row (y = 108–324)

| Button     | left      | top       | width | height | border-radius | z-index |
| --------- | --------- | --------- | ----- | ------ | -------------- | ------- |
| D-up       | 261.02px | 136px     | 54px  | 80px   | (svg polygon)  | 3       |
| D-down     | 261.02px | 216px     | 54px  | 80px   | (svg polygon)  | 3       |
| D-left     | 208px    | 189.02px  | 80px  | 54px   | (svg polygon)  | 3       |
| D-right    | 288px    | 189.02px  | 80px  | 54px   | (svg polygon)  | 3       |
| Y          | 451.25px | 136px     | 57.5px| 57.5px | 50%            | 3       |
| X          | 400px    | 187.25px  | 57.5px| 57.5px | 50%            | 3       |
| B          | 502.5px  | 187.25px  | 57.5px| 57.5px | 50%            | 3       |
| A          | 451.25px | 238.5px   | 57.5px| 57.5px | 50%            | 3       |
| LS boundary | 10px    | 120px     | 172px | 172px  | 50%            | 3       |
| RS boundary | 586px   | 120px     | 172px | 172px  | 50%            | 3       |

### Notable proportions

- **ABXY cross**: Each button is 57.5×57.5px. Arranged in a cross with 51.25px center-to-center spacing, inside a virtual 160×160px block centered at (480.25, 215.75). This is the classic Xbox diamond.
- **D-pad cross**: Each arm is 54×80 or 80×54. The D-pad center is (288, 215.02). The polygon geometry creates seamless junction.
- **LT/RT triggers**: 185px wide each — 24% of the controller width. 62px tall = 57% of the top row (108px).
- **LB/RB bumpers**: 92×62px — half the trigger width (without the pressure graph).
- **Analog sticks**: 172×172px boundary (66% of the LS outer). Inner knob = 66.67% of boundary = 114.67px. Knob travel = 28.67px (= 1/6 of the boundary).

---

## Spacing and visual rhythm

The overlay follows an **8px** ruler for primary spacing and **2px** for fine adjustments. All derived from the 4K base.

| Spacing                                | Pixels (at 768×324) | % of controller |
| -------------------------------------- | ------------------- | --------------- |
| Left margin of LT/LS                   | 10px                | 1.3%            |
| Right margin of RT/RS                   | 10px                | 1.3%            |
| Top of upper row                       | 23px                | 7.1%            |
| Top of lower row (D-pad, ABXY)         | 120-136px           | 37-42%          |
| Gap between LT and LB                  | 15px                | 1.95%           |
| Gap between LB and View                | 24px                | 3.13%           |
| Gap between View and Menu              | 0px (justaposed)    | 0%              |
| Gap between Menu and RB                | 22px                | 2.86%           |
| Gap between RB and RT                  | 16px                | 2.08%           |
| Gap between LS boundary and D-pad      | 26px                | 3.39%           |
| Gap between D-pad and ABXY (center to center) | ~135px       | 17.6%           |
| Gap between ABXY and RS boundary       | ~20px               | 2.6%            |
| Total D-pad height (top of D-up to bottom of D-down) | 160px | 49.4% |

### Balance principles

1. **Horizontal mirroring**: Everything on the left has an equivalent on the right. `left + width + right_partner_left = 768`.
2. **Equal spacing between pairs**: The 4 pairs (LT-LB, LB-View, Menu-RB, RB-RT) have gaps of 24, 22, 16, 15px — approximately equal (mean ~19px).
3. **Vertical center of D-pad** (215.02) ≈ **vertical center of ABXY** (215.75). Difference < 1px — optical alignment.
4. **Vertical center of analog sticks** (120 + 172/2 = 206) is ~9px above the D-pad/ABXY vertical center. Subtle, intentional, as the eye tends to perceive LS/RS as "behind" D-pad/ABXY.

---

## Z-index hierarchy

| z-index | Element                          | Reason                                       |
| ------- | --------------------------------- | -------------------------------------------- |
| 1       | SVG background `<rect>` (the overlay's base fill, controlled by `--color-bg` / `--bg-opacity`) | Always below everything |
| 2       | `.trigger-fill` (pressure fill, rendered on a sibling `<canvas>` positioned via `getBBox()`) | Above background, below button outlines |
| 3       | Common buttons (LB, RB, View, Menu, D-pad, ABXY, analog-boundary — the inner paths of each `<g id="...">` in the SVG) | Middle layer |
| 4       | LT, RT (triggers), analog-knob, arcade stick + trail (`<canvas>` for the ghost-glow trail) | Elements that move or have a visual press effect |
| 5       | `::after` / `<text>` labels (added by JS at runtime, centered on each button group) | Above all button content |

> Practical rule: elements that move or have a visual press effect should be at a higher z-index to not be obscured by static elements. In Track C, the SVG's paint order follows document order within each `<g>`, and the JS-managed `<canvas>` siblings sit above the SVG via CSS `z-index`.
>
> **Legacy (Track A/B):** the z-index 1 slot was held by `.controller-bg` (the PNG background image). Track C replaces it with the SVG's own background `<rect>` — no PNG, no `background-image` CSS.

---

## Animation timing

| Element                          | Animated property        | Duration | Easing     | Reason                          |
| --------------------------------- | ------------------------- | -------- | ---------- | ------------------------------ |
| Buttons (general)                | background, border-color, transform | 0.08s | ease-out   | Snappy, feels like a click     |
| SVG buttons (D-pad)              | fill, stroke              | 0.08s   | ease-out   | Same as others for consistency |
| Analog knob (movement)           | transform                 | 0.05s   | ease-out   | Faster than buttons to feel responsive |
| Analog knob (color)              | background, border-color  | 0.08s   | ease-out   | Same as buttons                |
| Trigger fill (width)             | width                     | 0.05s   | linear     | Linear = analog trigger feel    |
| Trigger fill (opacity)           | opacity                   | 0.05s   | linear     | Same as width for sync          |
| Trigger graph (canvas)            | (continuous, ~60fps)      | —       | —          | Updated every frame             |

### Principles

1. **Buttons at 80ms** — tactile "click" feel without seeming laggy.
2. **Analog at 50ms** — needs to be faster than buttons to not lag movement.
3. **Triggers at 50ms linear** — linear easing conveys a real trigger feel (no spring, goes straight).
4. **Trigger canvas runs on `requestAnimationFrame`** — draws continuous waves, but only costs CPU when there's an active gamepad (after `update()`).

---

## Planning a variant

Use this checklist to create a variant (arcade, DualSense, etc.) in the Track C architecture (SVG inline, see [VARIANTS.md](VARIANTS.md)):

1. **Decide the SVG canvas**:
   - Default viewBox is `0 0 768 324`. Keep it unless the layout truly needs a different shape (e.g., a taller fightstick).
   - If you change the viewBox, also update `--overlay-width` / `--overlay-height` in `index.html` `:root`.
2. **Define horizontal zones** (within the 768px width):
   - How many zones (left, center, right)?
   - What's the content of each?
3. **Plan proportions** (in SVG units, not % of a 3840×2160 base):
   - Use the rule of thirds for vertical division.
   - Keep equal spacing between equivalent pairs.
   - Mirror everything horizontally (left + width = 768 − right_partner_width).
4. **Draw in Inkscape** with each button as a named layer → `<g id="<button>">` (see [masks/controllers/README.md](masks/controllers/README.md)).
5. **Inline into `index.html`** as `<template id="svg-<layout>">` and register in the `CONTROLLERS` map (see [VARIANTS.md](VARIANTS.md) §"Translating a new SVG to HTML").
6. **Define z-index** for any sibling `<canvas>` effects (trigger graph, stick trail) — they sit above the SVG via CSS.
7. **Choose timings**: buttons = 80ms, analog = 50ms, triggers = 50ms linear. Only change with reason.
8. **Verify optical alignment**: vertical centers of equivalent groups (D-pad, ABXY, LS/RS) should match within ~1px.
9. **Document**: update this guide or create a `DESIGN_GUIDE_<LAYOUT>.md`. Add an entry to `docs/AUDIT.md`.

### Example: arcade variant

The arcade fightstick layout (`?controller=arcade`) is shipped as of `v1.2610.001` (open core, see [ROADMAP.md](ROADMAP.md)). Its SVG is at [masks/controllers/arcade.svg](masks/controllers/arcade.svg).
