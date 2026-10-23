# `masks/labels/` — Label Packs (Phase 5)

This folder contains **sprite sheets of button labels/icons** for the overlay. It's the planned implementation for Phase 5 of the project.

> ⚠️ **Status**: roadmap. Currently (Phases 1 and 2 implemented) labels come from `--label-*` CSS variables. This folder is a **model** for the future external sprite pack system.

---

## Table of Contents

- [What's here](#whats-here)
- [How a sprite pack works](#how-a-sprite-pack-works)
- [Using the ready-made model](#using-the-ready-made-model)
- [Creating your own pack](#creating-your-own-pack)
- [Full reference](#full-reference)

---

## What's here

```
masks/labels/
└── sprite_example.svg      ← ready-made model with 12 symbols (A, B, X, Y, LB, RB, LT, RT, LS, RS, View, Menu)
```

`sprite_example.svg` is an SVG containing multiple `<symbol>` elements (each with a unique `id`). To use in the overlay, you reference each symbol by ID via `<use href="#icon-a" />`.

See **[../../LABEL_PACKS_GUIDE.md](../../LABEL_PACKS_GUIDE.md)** for the full technique explanation (Technique A: background-position "XY" vs Technique B: SVG `<symbol>` + `<use>`).

---

## How a sprite pack works

A sprite pack is **a single SVG file** with all the overlay's icons. Instead of 12 separate files, 1 file.

Each icon is declared as a `<symbol>` inside the SVG:

```xml
<svg style="display: none">
    <symbol id="icon-a" viewBox="0 0 32 32">
        <text fill="currentColor">A</text>
    </symbol>
    <symbol id="icon-b" viewBox="0 0 32 32">
        <text fill="currentColor">B</text>
    </symbol>
    <!-- ...etc... -->
</svg>
```

In the overlay HTML, each button references its icon by ID:

```html
<div class="button a" id="a">
    <svg class="btn-icon"><use href="masks/labels/sprite.svg#icon-a" /></svg>
</div>
```

CSS controls the color (inherited from the button via `currentColor`):

```css
.btn-icon { fill: var(--button-color, var(--accent)); }
```

---

## Using the ready-made model

1. **Rename** `sprite_example.svg` to `sprite.svg` (or keep the name and adjust paths below).
2. **Edit `index.html`** — inside each button, add the `<svg><use>`:
   ```html
   <div class="button a" id="a">
       <svg class="btn-icon" viewBox="0 0 32 32" aria-hidden="true">
           <use href="masks/labels/sprite.svg#icon-a" />
       </svg>
   </div>
   ```
3. **Add the CSS**:
   ```css
   .btn-icon {
       position: absolute;
       top: 50%;
       left: 50%;
       width: 70%;
       height: 70%;
       transform: translate(-50%, -50%);
       fill: var(--button-color, var(--accent));
       pointer-events: none;
       z-index: 5;
   }
   ```
4. **(Optional) Disable CSS labels** — comment out the `.a::after { content: var(--label-a, "A"); }` rules in CSS, so there's no conflict between CSS text and SVG icon.
5. **Reload in OBS** — the overlay now uses the sprite's icons.

> To preview the sprite in isolation: uncomment the "Preview" block at the end of the `.svg` file and open in a browser. You'll see all 12 icons side by side in a grid.

---

## Creating your own pack

### Step 1 — Plan the IDs

Each `<symbol>` needs an ID matching the button:

| Overlay button    | Symbol ID           |
| ----------------- | -------------------- |
| A                 | `icon-a`             |
| B                 | `icon-b`             |
| X                 | `icon-x`             |
| Y                 | `icon-y`             |
| LB                | `icon-lb`            |
| RB                | `icon-rb`            |
| LT                | `icon-lt`            |
| RT                | `icon-rt`            |
| LS (knob)         | `icon-ls`            |
| RS (knob)         | `icon-rs`            |
| View              | `icon-view`          |
| Menu              | `icon-menu`          |

### Step 2 — Use consistent viewBox

All `<symbol>` elements should have the same viewBox (recommended `0 0 32 32`). This ensures consistent alignment inside buttons.

### Step 3 — Use `currentColor`

Always use `fill="currentColor"` or `stroke="currentColor"` on symbol elements. This makes the icon inherit the button's color via CSS:

```xml
<!-- ✅ Correct — inherits from CSS -->
<symbol id="icon-a" viewBox="0 0 32 32">
    <text fill="currentColor">A</text>
</symbol>

<!-- ❌ Wrong — fixed color, ignores --color-a -->
<symbol id="icon-a" viewBox="0 0 32 32">
    <text fill="#FFFFFF">A</text>
</symbol>
```

### Step 4 — Use paths/text, not images

Inside `<symbol>`, use pure SVG elements:
- `<text>` for letters (A, B, X, Y, LB, etc.)
- `<path>` for shapes (✕, ○, □, △, icons)
- `<rect>`, `<circle>`, `<polygon>` for geometric shapes
- `<g>` to group elements

**DO NOT use** `<image>` with external references inside the sprite — that breaks the single-sprite purpose.

### Step 5 — Test colors

After integrating, in the browser's DevTools (or by editing CSS), change:

```css
:root { --color-a: #FF0000; }
```

The A button's icon should turn red. If it didn't, you used `fill="#fixedColor"` instead of `fill="currentColor"` in some element of the `<symbol>`.

---

## Full reference

- **[../../LABEL_PACKS_GUIDE.md](../../LABEL_PACKS_GUIDE.md)** — full technique explanation (background-position "XY" vs SVG `<symbol>` + `<use>`), with pros/cons, code examples, and ready symbols for PlayStation, Nintendo, and system icons.
- **[../../CUSTOMIZATION_GUIDE.md](../../CUSTOMIZATION_GUIDE.md)** — Phases 1 and 2 guide (`--label-*` variables and `?icons=`).
- **[sprite_example.svg](sprite_example.svg)** — ready model with 12 Xbox default symbols + commented PlayStation block to copy.
