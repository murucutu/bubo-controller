# Label Packs Guide (Phase 5)

This guide explains how the "label packs" system planned for Phase 5 works: **a single file containing ALL buttons**, read by the overlay without needing to edit HTML for each new icon set.

You asked if it would be "a single SVG document containing ALL the buttons, used by the program with just XY directions to know where to read from". **Yes, that's exactly the concept** — it's called a **sprite sheet** (or *CSS sprite* / *SVG sprite*). There are **two techniques**. I'll explain both, provide a ready model, and tell you which to choose for this project.

---

## Table of Contents

- [What's a sprite sheet](#whats-a-sprite-sheet)
- [Technique A — Background-position (classic, "XY")](#technique-a--background-position-classic-xy)
- [Technique B — SVG `<symbol>` + `<use>` (modern)](#technique-b--svg-symbol--use-modern)
- [Which to choose for this project](#which-to-choose-for-this-project)
- [Ready-made model (sprite_example.svg)](#ready-made-model-sprite_examplesvg)
- [Integrating sprite packs into the overlay](#integrating-sprite-packs-into-the-overlay)
- [Creating your own sprite pack](#creating-your-own-sprite-pack)

---

## What's a sprite sheet

A sprite sheet is a single image (or SVG) containing multiple icons side by side, in a grid. Instead of loading 12 separate files (`a.png`, `b.png`, `x.png`, `y.png`, ...), you load 1 file and show only the piece you need per button.

**Advantages**:
- 1 HTTP request instead of N (faster in OBS).
- Easier to distribute (1 file = 1 pack).
- Packaging: swap pack = swap 1 file.

**Disadvantage**:
- Requires coordinates (which piece to show on which button).

There are two implementation approaches. The classic one is what you described ("XY directions"). The modern one uses symbolic references. Let's see both.

---

## Technique A — Background-position (classic, "XY")

This is **exactly what you imagined**: an image in a grid, and CSS uses `background-position: -Xpx -Ypx` to show only the right piece.

### How it works

Imagine a `sprite.png` with 4 icons in a 2×2 grid, each 32×32px:

```
+--------+--------+
|  A     |  B     |   <- row 0 (Y=0)
+--------+--------+
|  X     |  Y     |   <- row 1 (Y=32)
+--------+--------+
   col 0    col 1
   (X=0)    (X=32)
```

To show "A": `background-position: 0 0` (top-left corner).
To show "B": `background-position: -32px 0` (shift 32px left, revealing the 2nd icon).
To show "X": `background-position: 0 -32px`.
To show "Y": `background-position: -32px -32px`.

### Complete CSS

```css
.btn-label {
    width: 32px;
    height: 32px;
    background-image: url('sprite.png');
    background-repeat: no-repeat;
    /* default: show 1st icon */
    background-position: 0 0;
}

.btn-label.b { background-position: -32px 0; }
.btn-label.x { background-position: 0 -32px; }
.btn-label.y { background-position: -32px -32px; }
```

### Pros and cons

| Pro | Con |
| --- | --- |
| Simple to understand | **Doesn't allow dynamic color** — color is "baked" into the PNG |
| Works in any browser | Annoying pixel math (calculating X/Y for each icon) |
| Well supported | Doesn't scale perfectly on retina |
| Image can be PNG, JPG, SVG | Swapping packs = recalculating all coordinates |

> **For this project**: Technique A is **problematic** because it breaks the per-button color customization we built. If the sprite's "A" is red, you can't change it to blue without another sprite.

---

## Technique B — SVG `<symbol>` + `<use>` (modern)

This is **the recommended approach for this project**. Instead of a pixel grid, you create a single SVG containing multiple `<symbol>` elements (each with a unique `id`), and reference each symbol by ID using `<use>`.

### How it works

A `sprite.svg` file like this:

```xml
<svg xmlns="http://www.w3.org/2000/svg" style="display:none">
    <symbol id="icon-a" viewBox="0 0 32 32">
        <text x="16" y="22" text-anchor="middle" font-size="20" font-weight="700" fill="currentColor">A</text>
    </symbol>
    <symbol id="icon-b" viewBox="0 0 32 32">
        <text x="16" y="22" text-anchor="middle" font-size="20" font-weight="700" fill="currentColor">B</text>
    </symbol>
    <!-- ...etc... -->
</svg>
```

And in the HTML, you reference each symbol:

```html
<svg class="btn-label"><use href="sprite.svg#icon-a" /></svg>
```

### Why it's better for this project

| Pro | Why |
| --- | --- |
| **Dynamic color via CSS**     | Use `fill: currentColor` in the `<symbol>` and it inherits the button's color via CSS — preserves the `--color-*` customization |
| **Scales perfectly**           | SVG is vectorial, no pixelation at 4K |
| **No pixel math**              | Reference by `id` (`#icon-a`), not by coordinates |
| **Trivial editing**            | Open the `.svg` in a text editor, edit paths/text |
| **Same file**                  | 1 sprite.svg for all sets |
| **Easy swapping**              | Replace the file = swap the whole pack |

| Con | Mitigation |
| --- | ---------- |
| Requires inline SVG or external `<use>` | OBS CEF supports external `<use>` since CEF 50+ |
| No "preview" of the full image in the editor | Use an SVG viewer or open in a browser |
| Needs `fill="currentColor"` in symbols | Easy — just a habit to learn |

### Integration in the overlay

In `index.html`, inside each button:

```html
<div class="button a" id="a">
    <svg class="btn-icon" viewBox="0 0 32 32" aria-hidden="true">
        <use href="sprite.svg#icon-a" />
    </svg>
</div>
```

And in CSS:

```css
.btn-icon {
    position: absolute;
    top: 50%; left: 50%;
    width: 70%;
    height: 70%;
    transform: translate(-50%, -50%);
    fill: var(--button-color, var(--accent));  /* Inherits button color! */
    pointer-events: none;
    z-index: 5;
}
```

> The secret: `fill: var(--button-color, var(--accent))` on the external `<svg>`, and `fill="currentColor"` in the `<text>`/`<path>` inside the `<symbol>`. The external SVG overrides the fill declared in the symbol.

---

## Which to choose for this project

| Criterion                  | Technique A (PNG+XY) | Technique B (SVG+use) |
| -------------------------- | -------------------- | --------------------- |
| Per-button dynamic color   | ❌ color baked in    | ✅ `fill: currentColor` |
| 4K scaling                 | pixelates           | ✅ perfect             |
| Setup complexity           | simple              | medium                |
| OBS CEF support            | ✅ universal         | ✅ CEF 50+             |
| File size                  | larger (PNG)        | smaller (SVG text)    |
| Editing to create packs    | image editor        | text editor            |

**Recommendation**: use **Technique B (SVG with `<symbol>`)**. It preserves everything we built (per-button colors, B&W default, auto-scale). Technique A would be a regression — you'd lose color customization.

---

## Ready-made model (sprite_example.svg)

I've included a `masks/labels/sprite_example.svg` file in the project, with 4 example symbols (A, B, X, Y) written as SVG text. Use as a base for your own packs:

- **PlayStation pack**: replace the `A`/`B`/`X`/`Y` texts with ✕/○/□/△ paths.
- **Nintendo pack**: keep `A`/`B`/`X`/`Y` (same characters, layout already mapped via `?icons=nintendo`).
- **Arcade pack**: replace with `LP`/`MP`/`HP`/`LK`/`MK`/`HK` for fightstick.
- **Custom pack**: arbitrary SVG paths (logos, glyphs, etc.).

See [masks/labels/sprite_example.svg](masks/labels/sprite_example.svg) for the model.

---

## Integrating sprite packs into the overlay

When Phase 5 is implemented (not implemented yet — it's roadmap), the flow will be:

1. Create or download a `sprite.svg` containing `<symbol id="icon-a">`, `<symbol id="icon-b">`, etc.
2. Place it at `masks/labels/sprite.svg`.
3. In `index.html`, each button receives a `<svg><use>` inside.
4. (Future) URL param `?sprite=custom` to choose between multiple packs in `masks/labels/`.

The real integration will require refactoring the HTML to have `<svg><use>` inside each button. This is more work than Phases 1 and 2, which is why it's planned as Phase 5 (not as now).

> If you want to implement Phase 5 ahead of schedule, take the `sprite_example.svg` model, add `<svg class="btn-icon"><use href="masks/labels/sprite_example.svg#icon-a" /></svg>` inside each `.a`, `.b`, etc. in the HTML, and apply the `.btn-icon` CSS above.

---

## Creating your own sprite pack

### Step by step

1. **Decide the symbol IDs** — use `icon-a`, `icon-b`, `icon-x`, `icon-y`, `icon-lb`, `icon-rb`, `icon-lt`, `icon-rt`, `icon-ls`, `icon-rs`, `icon-view`, `icon-menu` (matching the project's CSS classes).

2. **Create the SVG file** — use `sprite_example.svg` as a template. Each `<symbol>` needs:
   - `id="icon-X"`
   - `viewBox="0 0 32 32"` (same viewBox for consistency)
   - Content: `<path>`, `<text>`, `<circle>`, etc.
   - Use `fill="currentColor"` on filled elements (to inherit button color).

3. **Test in isolation** — open the `.svg` in a browser. You won't see anything (because `<symbol>` needs `<use>` to render). To preview, add at the end of the file:
   ```xml
   <svg width="100" height="100"><use href="#icon-a" /></svg>
   ```
   Remove before publishing.

4. **Test in the overlay** — replace `sprite_example.svg` in `masks/labels/` with your file (keep the name), reload in OBS.

5. **Validate colors** — with the overlay open, change `--color-a: red` in DevTools. The `A` should turn red. If it didn't, you used `fill="#FFFFFF"` instead of `fill="currentColor"` in the `<symbol>`.

### Ready-to-copy symbols

#### PlayStation — Cross (✕) as path

```xml
<symbol id="icon-a" viewBox="0 0 32 32">
    <path d="M8 8 L24 24 M24 8 L8 24" stroke="currentColor" stroke-width="3" fill="none" stroke-linecap="round" />
</symbol>
```

#### PlayStation — Circle (○) as path

```xml
<symbol id="icon-b" viewBox="0 0 32 32">
    <circle cx="16" cy="16" r="9" stroke="currentColor" stroke-width="3" fill="none" />
</symbol>
```

#### PlayStation — Square (□) as path

```xml
<symbol id="icon-x" viewBox="0 0 32 32">
    <rect x="8" y="8" width="16" height="16" stroke="currentColor" stroke-width="3" fill="none" />
</symbol>
```

#### PlayStation — Triangle (△) as path

```xml
<symbol id="icon-y" viewBox="0 0 32 32">
    <path d="M16 8 L26 24 L6 24 Z" stroke="currentColor" stroke-width="3" fill="none" stroke-linejoin="round" />
</symbol>
```

#### Xbox — A/B/X/Y letters (text)

```xml
<symbol id="icon-a" viewBox="0 0 32 32">
    <text x="16" y="22" text-anchor="middle" font-size="18" font-weight="700" font-family="sans-serif" fill="currentColor">A</text>
</symbol>
```

(Repeat for B/X/Y, swapping the `<text>` content.)

#### Hamburger menu (placeholder for Menu)

```xml
<symbol id="icon-menu" viewBox="0 0 32 32">
    <path d="M6 10 H26 M6 16 H26 M6 22 H26" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" />
</symbol>
```

#### View icon (two overlapping rectangles)

```xml
<symbol id="icon-view" viewBox="0 0 32 32">
    <rect x="6" y="8" width="14" height="12" stroke="currentColor" stroke-width="2" fill="none" />
    <rect x="12" y="12" width="14" height="12" stroke="currentColor" stroke-width="2" fill="none" />
</symbol>
```

---

## Final tip

Sprite sheets solve a specific problem: **distributing an icon pack as 1 file**. If you only need 4 labels (A/B/X/Y), the sprite overhead is bigger than the benefit — use the `--label-*` variables (Phase 1) or `?icons=` (Phase 2).

Sprites are worth it when:
- You have 10+ icons with complex SVG paths (can't write as text).
- You want to distribute "themes" as disposable packs (Xbox pack, PS pack, retro pack, neon pack).
- The community will contribute packs without editing the HTML.

For the current overlay, **Phases 1 and 2 cover 90% of use cases**. Sprite sheets (Phase 5) are for when the project grows and needs external packs.
