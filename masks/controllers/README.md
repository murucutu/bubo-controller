# `masks/controllers/` — Controller SVG Blueprints

This folder contains the **SVG blueprints** that define each controller layout. These SVGs are the source of truth for button positions, sizes, and shapes.

---

## Table of Contents

- [What's here](#whats-here)
- [How it works](#how-it-works)
- [SVG structure requirements](#svg-structure-requirements)
- [Creating a new controller layout](#creating-a-new-controller-layout)
- [Inkscape workflow](#inkscape-workflow)
- [Naming conventions](#naming-conventions)

---

## What's here

```
masks/controllers/
├── xbox.svg      # Xbox-style gamepad layout
├── arcade.svg    # Arcade fightstick (Viewlix) layout
└── README.md     # This file
```

Each SVG is a self-contained 768×324 file with named layers for every button.

---

## How it works

1. **You draw** the controller in Inkscape/Illustrator, with each button on its own named layer.
2. **The maintainer** runs a script to inline the SVG into `index.html` as a `<template>` element.
3. **At runtime**, JS clones the appropriate SVG template into the DOM based on `?controller=`.
4. **JS finds** each `<g id="X">` and animates it (adds/removes `.pressed` class).
5. **CSS controls** the visual states (fill, opacity, transitions).

The SVG is both the **blueprint** (documentation of positions) and the **runtime art** (what the user sees). This is "Track C" from [VARIANTS.md](../../VARIANTS.md).

---

## SVG structure requirements

Each SVG must follow this structure:

```xml
<svg viewBox="0 0 768 324" xmlns="http://www.w3.org/2000/svg">
    <g id="Prancheta1">
        <!-- Background rect (768×324) — fill controlled by CSS -->
        <rect x="0" y="0" width="768" height="324"/>

        <!-- Each button is a <g> with a unique id -->
        <g id="a" transform="matrix(...)">
            <!-- Outer shape (the 2pt stroke gap) — CSS sets fill:none -->
            <circle cx="..." cy="..." r="..."/>
            <!-- Inner shape (the button fill) — CSS controls fill and opacity -->
            <path d="..." style="fill:white;"/>
        </g>

        <!-- More buttons... -->
    </g>
</svg>
```

### Key requirements

1. **ViewBox**: `0 0 768 324` (mandatory — matches the overlay dimensions).
2. **Root group**: `<g id="Prancheta1">` containing the background rect and all buttons.
3. **Background rect**: `<rect x="0" y="0" width="768" height="324"/>` as the first child.
4. **Button groups**: Each button is a `<g id="BUTTON_ID">` with a `transform` attribute.
5. **Two-path structure**: Each button group has exactly 2 children:
   - First child: the outer shape (creates the 2pt stroke gap — CSS sets `fill: none`)
   - Second child: the inner shape (the fill — CSS controls color and opacity)
6. **Layer IDs**: Must match the [naming conventions](#naming-conventions) below.

### What JS does to the SVG at runtime

When the SVG is cloned into the DOM, JS:
1. Strips all inline `style` attributes (removes `fill:white` etc.)
2. Removes all `fill` attributes from paths/circles/rects
3. Lets CSS control all fills via `--button-color` and `fill-opacity`
4. Adds `<text>` elements for labels at the center of each button group

---

## Creating a new controller layout

### Step 1 — Draw in Inkscape

1. Create a new document: **768 × 324 px**.
2. Draw the background rect: `0,0` to `768,324`.
3. Draw each button as a circle/path on its own layer.
4. **Rename each layer** to the button ID (e.g., `a`, `lb`, `left_joy`).
5. Use the 2-path structure: an outer shape + an inner shape per button.
6. Set `fill:white` on the inner path (will be stripped by JS at runtime).

### Step 2 — Export

1. Save as **SVG** (Inkscape: File → Save As → SVG).
2. The exported file should have `<g id="...">` elements matching your layer names.

### Step 3 — Add to the project

1. Place the SVG in `masks/controllers/your_controller.svg`.
2. Ask the maintainer to inline it into `index.html` as a `<template id="svg-your_controller">`.
3. Add the controller to the `CONTROLLERS` map in the JS:
   ```js
   const CONTROLLERS = Object.freeze({
       xbox: 'svg-xbox',
       arcade: 'svg-arcade',
       your_controller: 'svg-your_controller'  // new
   });
   ```
4. Test with `?controller=your_controller` in the URL.

---

## Inkscape workflow

### Setup

1. **File → Document Properties**: set width=768, height=324, units=px.
2. **Grid**: enable grid, spacing 1px, for precise positioning.
3. **Snapping**: enable snap to grid.

### Drawing buttons

1. **Create a layer** for each button (Layer → Add Layer).
2. **Name the layer** with the button ID (e.g., `a`, `lb`, `left_joy`).
3. **Draw the outer shape** (circle, rect, or path) — this creates the 2pt stroke gap.
4. **Draw the inner shape** (slightly smaller, same center) — set `fill:white`.
5. **Group both shapes** (Ctrl+G) — the group gets the layer ID.

### Tips

- Use **Inkscape's Align and Distribute** dialog to center the inner shape within the outer.
- The gap between outer and inner should be ~2px (the "stroke" width).
- For circles: draw outer with radius R, inner with radius R−2.
- For rects: draw outer at size W×H, inner at (W−4)×(H−4), centered.

### Export

1. **File → Save As → Plain SVG** (not "Inkscape SVG" — avoids extra metadata).
2. The exported SVG should have clean `<g id="...">` elements.
3. Verify by opening the SVG in a text editor — check that each button has the correct `id`.

---

## Naming conventions

Use these IDs for button layers. JS maps them to the Standard Gamepad API.

### Face buttons

| ID | Gamepad button index | Default label |
| -- | -------------------- | ------------- |
| `a` | 0 | A / ✕ / B |
| `b` | 1 | B / ○ / A |
| `x` | 2 | X / □ / Y |
| `y` | 3 | Y / △ / X |

### Shoulder buttons

| ID | Gamepad button index | Default label |
| -- | -------------------- | ------------- |
| `lb` | 4 | LB / L1 / L |
| `rb` | 5 | RB / R1 / R |
| `lt` | 6 | LT / L2 / ZL |
| `rt` | 7 | RT / R2 / ZR |

### System buttons

| ID | Gamepad button index | Default label |
| -- | -------------------- | ------------- |
| `view` | 8 | (View/Share) |
| `menu` | 9 | (Menu/Options) |
| `l3` | 10 | L3 |
| `r3` | 11 | R3 |

### D-pad (Xbox only)

| ID | Gamepad button index |
| -- | -------------------- |
| `up` | 12 |
| `down` | 13 |
| `left` | 14 |
| `right` | 15 |

### Analog sticks (Xbox)

| ID | Element |
| -- | ------- |
| `left_joy_reach` | LS boundary ring |
| `left_joy` | LS knob |
| `right_joy_reach` | RS boundary ring |
| `right_joy` | RS knob |

### Arcade stick (Arcade only)

| ID | Element |
| -- | ------- |
| `joy_reach` | Stick boundary ring |
| `joy` | Stick knob |

### Notes

- The `_reach` suffix is special — JS applies lower opacity (15%) to these elements.
- D-pad is only used in the Xbox layout. The arcade layout captures D-pad input via the stick.
- L3/R3 in the arcade layout are separate round buttons (not stick-click).
- L3/R3 in the Xbox layout are the stick-click (pressing LS/RS knobs).
