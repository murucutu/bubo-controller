# Customization Guide

This guide covers everything you can visually adjust in the overlay: colors (global and per-button), button shapes, custom backgrounds, and ready-to-copy style presets.

All changes are made by editing `:root` at the top of the `<style>` block in `index.html`. No build, no compilation — just edit and reload the page in OBS.

---

## Table of Contents

- [Colors](#colors)
  - [Global colors](#global-colors)
  - [Per-button colors](#per-button-colors)
  - [Ready-made palettes](#ready-made-palettes)
- [Labels (button text)](#labels-button-text)
  - [Default label set (Xbox)](#default-label-set-xbox)
  - [Icon sets via URL (`?icons=`)](#icon-sets-via-url-icons)
  - [Custom label per button](#custom-label-per-button)
  - [Label styling (font, size)](#label-styling-font-size)
  - [When the label becomes invisible](#when-the-label-becomes-invisible)
- [Button shapes](#button-shapes)
  - [ABXY (border-radius)](#abxy-border-radius)
  - [D-pad (SVG polygons)](#d-pad-svg-polygons)
  - [Analog sticks](#analog-sticks)
  - [Triggers (corners)](#triggers-corners)
- [Custom backgrounds (masks/)](#custom-backgrounds-masks)
- [Stroke, opacity, and analog stick travel](#stroke-opacity-and-analog-stick-travel)
- [Light vs dark themes](#light-vs-dark-themes)

---

## Colors

### Global colors

Two variables control the base theme:

```css
:root {
    --accent: #FFFFFF;     /* fallback color for any button without --color-X defined */
    --color-bg: #000000;   /* background tint reference (not used at runtime, just documentary) */
}
```

- `--accent` is the default applied to all buttons that don't have their own color set.
- `--color-bg` is not used in runtime CSS (the background comes from `masks/bg_dark.png`). It exists only to document the intended palette.

To change everything to one color, just swap `--accent`. To give individual buttons specific colors, use the per-button variables (below).

### Per-button colors

Each button has its own variable, defaulting to `--accent`:

| Button(s)            | CSS variable        |
| -------------------- | ------------------- |
| A, B, X, Y           | `--color-a`, `--color-b`, `--color-x`, `--color-y` |
| LB, RB (bumpers)     | `--color-lb`, `--color-rb` |
| LT, RT (triggers)    | `--color-lt`, `--color-rt` |
| LS, RS (analog sticks) | `--color-ls`, `--color-rs` |
| D-pad (4 directions) | `--color-dpad` (one color for all 4) |
| View                 | `--color-view`      |
| Menu                 | `--color-menu`      |

#### Example 1 — Official Xbox layout (colored ABXY)

```css
:root {
    --color-a: #2EBD32;   /* green */
    --color-b: #E3121A;   /* red */
    --color-x: #0066CC;   /* blue */
    --color-y: #F2BB1D;   /* yellow */
}
```

#### Example 2 — PlayStation layout (symbols, no color)

Keep `--color-a`…`--color-y` as `var(--accent)` (default white). The visual difference between Xbox and PlayStation is in the shapes, not colors. See [Button shapes](#button-shapes) below.

#### Example 3 — Neon theme

```css
:root {
    --accent: #00FF7F;          /* neon green */
    --color-lt: #FF00FF;        /* magenta on LT */
    --color-rt: #00FFFF;        /* cyan on RT */
    --color-a: #FFD700;         /* gold on A */
}
```

#### Example 4 — Pastel theme

```css
:root {
    --accent: #F5E6E6;          /* light pink */
    --color-a: #FFB3BA;
    --color-b: #BAE1FF;
    --color-x: #B5EAD7;
    --color-y: #FFE5B4;
}
```

#### Example 5 — Red on A only (highlight)

```css
:root {
    --color-a: #FF0000;   /* everything white, A red */
}
```

### How the color is applied

Each button receives `--button-color: var(--color-X)` via CSS. Borders, fill when pressed, trigger gradients, glow shadow, and even the trigger waveform canvas — everything reads the button's color.

Translucency is done via `color-mix(in srgb, <color> X%, transparent)` in CSS, and via `rgba()` in the canvas (reads the color once on init in `parseColorToRgb`).

> **Tip**: since the trigger canvas reads the color only on init, if you change the color via DevTools live, the canvas won't update. Reload the page to apply.

---

## Labels (button text)

Each button can display text (or a Unicode symbol) in the center of its shape. The text is injected via `::after` and controlled by `--label-*` CSS variables.

### Default label set (Xbox)

By default, the overlay shows Xbox labels:

```css
:root {
    --label-a: "A";  --label-b: "B";  --label-x: "X";  --label-y: "Y";
    --label-lb: "LB";  --label-rb: "RB";
    --label-lt: "LT";  --label-rt: "RT";
    --label-ls: "";  --label-rs: "";     /* knobs usually without text */
    --label-view: "";  --label-menu: ""; /* SVG icons coming in Phase 3 */
}
```

### Icon sets via URL (`?icons=`)

Swap the entire set without editing HTML — add `?icons=SET` to the OBS Browser Source URL:

| URL                                              | ABXY labels        | Shoulder buttons |
| ------------------------------------------------ | ------------------ | ----------------- |
| `index.html`                                     | A B X Y (Xbox)     | LB RB LT RT       |
| `index.html?icons=xbox`                          | A B X Y (Xbox)     | LB RB LT RT       |
| `index.html?icons=playstation`                   | ✕ ○ □ △ (PS symbols) | L1 R1 L2 R2      |
| `index.html?icons=nintendo`                       | B A Y X (Switch layout) | L R ZL ZR    |
| `index.html?icons=none`                           | (no labels)       | (no labels)        |

#### POSITION → label mapping

Important: labels represent **what's physically on the controller**, mapped by **position** in the overlay (not by Gamepad API index).

| Position in overlay | Xbox label | PlayStation label | Nintendo label |
| ------------------ | ---------- | ----------------- | --------------- |
| Bottom (A slot)     | A          | ✕ (Cross)         | B               |
| Right  (B slot)     | B          | ○ (Circle)        | A               |
| Left   (X slot)     | X          | □ (Square)        | Y               |
| Top    (Y slot)     | Y          | △ (Triangle)      | X               |

> On Nintendo controllers plugged into a PC, the Gamepad API may swap indices (A might become B). This is a known browser issue, not the overlay. The visual label reflects position in the drawing, not the physical button pressed.

### Custom label per button

For your own labels (e.g., mapping to in-game actions):

```css
:root {
    --label-a: "JUMP";
    --label-b: "SHOOT";
    --label-x: "GUARD";
    --label-y: "DASH";
}
```

Or single characters:

```css
:root {
    --label-a: "1";
    --label-b: "2";
    --label-x: "3";
    --label-y: "4";
}
```

You can combine `?icons=` with CSS overrides — but note the URL always wins. For permanent CSS overrides, **don't** use `?icons=` in the OBS URL.

### Label styling (font, size)

```css
:root {
    --label-font: "Inter", "Segoe UI", system-ui, sans-serif;
    --label-size: 26px;            /* ABXY */
    --label-size-small: 18px;       /* LB/RB/LT/RT/View/Menu */
    --label-weight: 700;
}
```

- `--label-size` applies to ABXY (larger buttons, 57.5×57.5px).
- `--label-size-small` applies to LB/RB/LT/RT (smaller buttons, 92×62 and 185×62).
- To scale all up, adjust both.

#### Example — mono font labels, 30px size

```css
:root {
    --label-font: "JetBrains Mono", "Fira Code", monospace;
    --label-size: 30px;
    --label-size-small: 22px;
}
```

### When the label becomes invisible

Watch out for combinations that erase the text:

- **Button colored X with label in the same color X**: text disappears against the transparent background. Use a different label color via `--label-*` color override (advanced).
- **Button pressed**: the fill becomes `var(--button-color)` (e.g., red) and the label becomes `var(--color-bg)` (e.g., black) automatically. If `--color-bg` is wrong, the pressed label disappears.
- **`?icons=none`**: removes all labels (just the button shapes).

> In light themes (`--accent: #000`, `--color-bg: #FFF`), the pressed label is white on black fill — visible. In dark themes (`--accent: #FFF`, `--color-bg: #000`), the pressed label is black on white fill — also visible.

---

## Button shapes

Each button family uses a different drawing technique. Below is what each controls and how to change it.

### ABXY (border-radius)

A/B/X/Y buttons are `<div>` elements made circular via `border-radius: 50%`. To change the shape, edit the button rule:

```css
.a { border-radius: 50%; }   /* circle (default) */
.a { border-radius: 0; }    /* square */
.a { border-radius: 12px; } /* rounded corners */
.a { border-radius: 8px; }  /* soft corners */
.a { border-radius: 50% 0 50% 0; }  /* leaf / petal */
.a {
    border-radius: 0;
    clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);  /* diamond */
}
```

Applies the same to `.b`, `.x`, `.y`.

### D-pad (SVG polygons)

The 4 D-pad directions are SVGs with `<polygon>` in `viewBox="-1 -1 102 102"` (space so the stroke isn't clipped). The points form trapezoidal arrows.

#### Current points (reference)

| Direction | Polygon points                              |
| --------- | -------------------------------------------- |
| Up        | `0,0  100,0  100,67.44  50,100  0,67.44`     |
| Down      | `50,0  100,32.56  100,100  0,100  0,32.56`   |
| Left      | `0,0  67.44,0  100,50  67.44,100  0,100`     |
| Right     | `32.56,0  100,0  100,100  32.56,100  0,50`   |

#### Variation 1 — Square D-pad (no arrow)

```html
<polygon points="0,0 100,0 100,100 0,100" />
```

#### Variation 2 — Diamond D-pad (Pinball-style)

```html
<polygon points="50,0 100,50 50,100 0,50" />
```

#### Variation 3 — Thick cross D-pad

Increase `width`/`height` to `80×80` (instead of 54×80) and use a diamond polygon. Adjust `left`/`top` to keep alignment at the D-pad center.

> For complex layouts, see [DESIGN_GUIDE.md](DESIGN_GUIDE.md) — it explains the grid and spacing for planning a fully new D-pad.

### Analog sticks

Each analog stick is an `.analog-boundary` (outer circle) + `.analog-knob` (inner circle that moves).

```css
.analog-boundary { border-radius: 50%; }       /* default circular */
.analog-knob      { border-radius: 50%; }      /* default circular */
```

#### Variation 1 — Square knob

```css
.analog-knob { border-radius: 0; }
```

#### Variation 2 — Square boundary, circular knob (arcade-style)

```css
.analog-boundary { border-radius: 8px; }
.analog-knob      { border-radius: 50%; }
```

#### Knob size

```css
.analog-knob {
    width: 75%;     /* default 66.67% — increase to fill more of the boundary */
    height: 75%;
}
```

### Triggers (corners)

LT/RT have `border-radius: 12px` by default. Variations:

```css
.lt, .rt { border-radius: 0; }      /* pure rectangle */
.lt, .rt { border-radius: 999px; }  /* pill */
.lt, .rt { border-radius: 12px 12px 0 0; }  /* only top corners rounded */
```

---

## Custom backgrounds (masks/)

The overlay background is a PNG file called `masks/bg_dark.png` (3840×2160, 16:9 format). The CSS crops the central region (768×324) and uses it as the controller backdrop.

To swap the background:

1. **Simply replace the file** — create a new `masks/bg_dark.png` (different name, edit `background-image` in CSS). Keep size 3840×2160 and the controller drawn in the central region (`x=1536..2304, y=1728..2052`).
2. **Change the crop** — adjust `--crop-x` and `--crop-y` in `:root` to show a different region of the image.
3. **Change the base size** — if your image is 1920×1080 instead of 3840×2160, adjust `--base-width`/`--base-height` and recalculate `--crop-x`/`--crop-y` proportionally.

See **[masks/README.md](masks/README.md)** for a complete guide on drawing a background from scratch, including the reference grid and where each button should land on the base image.

### Light background (light theme)

For a light theme:

1. Create `masks/bg_light.png` with a white background instead of black.
2. In CSS, swap `--accent: #FFFFFF` to `--accent: #000000` (black on white).
3. Update `background-image: url('masks/bg_light.png')`.

You can keep both in the project and comment/uncomment as needed.

---

## Stroke, opacity, and analog stick travel

```css
:root {
    --stroke: 5px;             /* border thickness (default 5px at 1280x720 scale) */
    --opacity: 0.80;           /* overall overlay opacity (0 = invisible, 1 = opaque) */
    --analog-max-offset: 28.6667px;  /* max knob travel within the boundary */
}
```

### When to increase `--stroke`

- On 4K streams (3840×2160), the overlay scales 5×. A `--stroke: 5px` becomes 25px visually — usually fine.
- For smaller streams (720p), reduce to 3px.

### When to adjust `--analog-max-offset`

- If the knob "jumps" outside the boundary when pushing the stick to the extreme, increase the value.
- If the knob barely moves, decrease it.
- Default `28.6667px` comes from the boundary size (172px) minus the knob (66.67% of 172 = 114.67px), divided by 4 for comfortable visual travel.

---

## Light vs dark themes

The project ships with `--accent: #FFFFFF` (white on dark background). To invert:

### Dark theme (default)

```css
:root {
    --accent: #FFFFFF;
    --color-bg: #000000;
}
/* masks/bg_dark.png */
```

### Light theme

```css
:root {
    --accent: #000000;
    --color-bg: #FFFFFF;
}
/* masks/bg_light.png (create a light version of the background) */
```

### Adaptive theme (light or dark based on prefers-color-scheme)

```css
:root {
    --accent: #FFFFFF;
    --color-bg: #000000;
}

@media (prefers-color-scheme: light) {
    :root {
        --accent: #000000;
        --color-bg: #FFFFFF;
    }
    .controller-bg {
        background-image: url('masks/bg_light.png');
    }
}
```

> OBS Browser Source inherits the system color scheme. If your OS is in dark mode, the overlay is dark; in light mode, it's light.
