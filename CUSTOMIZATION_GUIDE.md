# Customization Guide

This guide covers everything you can visually adjust in the overlay: colors, opacity system, stick trail tuning, button labels, and how to create custom controller SVGs.

All changes are made by editing `:root` at the top of the `<style>` block in `index.html`. No build, no compilation — just edit and reload the page in OBS.

---

## Table of Contents

- [Controller Selection](#controller-selection)
- [Colors](#colors)
  - [Global colors](#global-colors)
  - [ABXY color schemes (`?color=`)](#abxy-color-schemes-color)
  - [Per-button colors](#per-button-colors)
- [Opacity System](#opacity-system)
- [Button Labels (`?icons=`)](#button-labels-icons)
- [Stick Trail (Arcade only)](#stick-trail-arcade-only)
- [Light vs Dark Themes](#light-vs-dark-themes)

---

## Controller Selection

Choose which controller layout to display via URL:

```
index.html?controller=xbox      # Xbox-style gamepad (default)
index.html?controller=arcade    # Arcade fightstick (Viewlix layout)
```

Each layout is driven by an SVG blueprint in `masks/controllers/`. The SVG is inlined in `index.html` as a `<template>` element; JS clones it into the DOM at runtime and animates the buttons.

See [masks/controllers/README.md](masks/controllers/README.md) for how to create new controller layouts.

---

## Colors

### Global colors

Two variables control the base theme:

```css
:root {
    --accent: #FFFFFF;     /* global color — used for all buttons and background */
    --color-bg: #000000;   /* label text color (for contrast against buttons) */
}
```

- `--accent` is the default color applied to all buttons and the background rect.
- `--color-bg` is used for label text fill, ensuring contrast against the button fill.

To invert to a light theme (dark buttons on white background):

```css
:root {
    --accent: #000000;
    --color-bg: #FFFFFF;
}
```

### ABXY color schemes (`?color=`)

Swap the ABXY color set via URL — no HTML editing needed. Colors are applied to button **outlines** (inner path) and to the pressed fill. The arcade stick trail also uses these colors in gradient (see [Stick Trail](#stick-trail-arcade-only)).

| URL | A | B | X | Y |
| --- | --- | --- | --- | --- |
| `index.html` (default) | white | white | white | white |
| `?color=xbox` | green `#4CAF50` | red `#E5342A` | blue `#0E7DC0` | amber `#F2C200` |
| `?color=playstation` | blue `#2E6DB4` (✕ Cross) | red `#E5342A` (○ Circle) | pink `#FF69F8` (□ Square) | green `#3EE3A1` (△ Triangle) |

#### Official color sources

**Xbox** (verified from Xbox 360 / Xbox One controller):
- A = green `#4CAF50` — Material Design green 500, close to Xbox's signature green
- B = red `#E5342A` — Xbox red
- X = blue `#0E7DC0` — Xbox blue
- Y = amber `#F2C200` — Xbox amber/yellow

**PlayStation** (verified from PS1/PS4 designer Teiyu Goto's official statement):
- Cross (✕) = blue `#2E6DB4` — "yes" decision color
- Circle (○) = red `#E5342A` — "no" decision color
- Square (□) = pink `#FF69F8` — represents a piece of paper (menu items)
- Triangle (△) = green `#3EE3A1` — represents viewpoint/direction (head)

> Source: Teiyu Goto, Sony designer, in interviews about the original PlayStation controller (1994). The colors have remained consistent across PS1, PS2, PS3, PS4, and PS5.

The `?color=` parameter only sets the 4 face button colors (A/B/X/Y). All other buttons (LB, RB, LT, RT, D-pad, sticks) remain `--accent` (white by default).

### Per-button colors

Each button has its own CSS variable, defaulting to `--accent`:

| Button(s)            | CSS variable        |
| -------------------- | ------------------- |
| A, B, X, Y           | `--color-a`, `--color-b`, `--color-x`, `--color-y` |
| LB, RB (bumpers)     | `--color-lb`, `--color-rb` |
| LT, RT (triggers)    | `--color-lt`, `--color-rt` |
| LS, RS (analog sticks) | `--color-ls`, `--color-rs` |
| Arcade stick         | `--color-stick` |
| D-pad (4 directions) | `--color-dpad` (one color for all 4) |
| View                 | `--color-view`      |
| Menu                 | `--color-menu`      |
| L3, R3 (stick click) | `--color-l3`, `--color-r3` |

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

---

## Opacity System

The overlay uses a **layered opacity system** that creates visual depth:

```css
:root {
    --bg-opacity: 0.4;              /* background rect fill opacity */
    --button-opacity: 0.4;          /* unpressed button fill opacity */
    --button-pressed-opacity: 1;    /* pressed button fill opacity (100%) */
    --boundary-opacity: 0.15;       /* analog stick boundary rings */
}
```

### How it works

1. **Background rect** (768×324): filled with `--accent` at 40% opacity.
2. **All buttons** (including analog stick knobs): filled with their color at 40% opacity when unpressed.
3. **Where button overlaps background**: alpha blending produces ~64% opacity (1 − (1−0.4)×(1−0.4) = 0.64). This makes buttons appear denser than the background alone.
4. **Pressed buttons**: fill jumps to 100% opacity — bright flash on press.
5. **Boundary rings** (analog stick outer circles): at 15% opacity — subtle, doesn't compete with buttons.

### Tuning the opacity

For a more subtle look (lighter overlay):

```css
:root {
    --bg-opacity: 0.25;
    --button-opacity: 0.25;
}
```

For a more visible overlay (darker buttons):

```css
:root {
    --bg-opacity: 0.5;
    --button-opacity: 0.6;
}
```

> The combined overlap opacity is approximately `bg + button − (bg × button)`. With 0.4 + 0.4: 0.4 + 0.4 − 0.16 = 0.64. With 0.5 + 0.6: 0.5 + 0.6 − 0.30 = 0.80.

### LT/RT trigger wave effect (Xbox only)

The Xbox layout preserves the original trigger wave/gradient effect. LT and RT have:
- A gradient fill that grows from inside-out based on pressure
- A canvas waveform overlay that reacts to pressure in real time

These are overlaid on top of the SVG trigger shapes. The wave color follows `--color-lt` / `--color-rt`. This effect is **not** affected by the opacity system — it has its own opacity (72% unpressed, 100% pressed).

---

## Button Labels (`?icons=`)

Swap the label text set via URL:

| URL | ABXY labels | Shoulder labels |
| --- | --- | --- |
| `?icons=xbox` (default) | A B X Y | LB RB LT RT |
| `?icons=playstation` | ✕ ○ □ △ | L1 R1 L2 R2 |
| `?icons=nintendo` | B A Y X (Switch layout) | L R ZL ZR |
| `?icons=none` | (no labels) | (no labels) |

Labels are rendered as SVG `<text>` elements, positioned at the center of each button by JS. The label text color uses `--color-bg` for contrast.

### Custom labels

Edit the `--label-*` variables in `:root`:

```css
:root {
    --label-a: "JUMP";
    --label-b: "SHOOT";
    --label-x: "GUARD";
    --label-y: "DASH";
}
```

### Label font and size

```css
:root {
    --label-font: "Inter", "Segoe UI", system-ui, sans-serif;
    --label-size: 22px;            /* ABXY */
    --label-size-small: 16px;       /* LB/RB/LT/RT/L3/R3/View/Menu */
    --label-weight: 700;
}
```

---

## Stick Trail (Arcade only)

The arcade layout features a ghost-glow stick trail (inspired by Arc System Works fighting games). When the stick moves, a trail of fading circles follows behind the knob.

### Trail colors

By default (no `?color=` parameter), the trail is **white**. When `?color=xbox` or `?color=playstation` is set, the trail uses a **4-color gradient** following the ABXY sequence:

| `?color=` | Trail colors (newest → oldest) | Meaning |
| --- | --- | --- |
| (none) | white → white → white → white | Default neutral |
| `xbox` | green → red → blue → amber | A → B → X → Y |
| `playstation` | blue → red → pink → green | Cross → Circle → Square → Triangle |

The newest trail position (closest to the knob) uses the first color (A/Cross), and the oldest position (farthest) uses the fourth color (Y/Triangle). The trail is divided into 4 equal segments, each painted in its respective color.

### Trail parameters

```css
:root {
    --stick-trail-length: 30;       /* number of trail frames (higher = longer trail) */
    --stick-trail-fade: 0.04;       /* fade rate per frame (lower = softer/longer fade) */
    --stick-trail-radius: 12;       /* trail circle radius in px (1/3 of knob radius) */
    --stick-trail-blur: 12px;       /* ghost glow blur in px */
    --stick-max-offset: 72px;       /* max knob travel from center */
}
```

### Tuning the trail

**Softer, longer trail** (more ethereal):
```css
:root {
    --stick-trail-length: 40;
    --stick-trail-fade: 0.02;
    --stick-trail-blur: 16px;
}
```

**Sharper, shorter trail** (more precise):
```css
:root {
    --stick-trail-length: 15;
    --stick-trail-fade: 0.10;
    --stick-trail-blur: 6px;
}
```

**Bigger trail circles** (more visible):
```css
:root {
    --stick-trail-radius: 18;       /* default 12; knob radius is 36 */
}
```

### How the trail works

1. Each animation frame, the knob's current position is recorded.
2. The trail stores the last N positions (N = `--stick-trail-length`).
3. A canvas draws:
   - A connecting line (motion blur effect) with ghost glow
   - N circles at 1/3 the knob size, with alpha increasing from old to new
4. Only the older positions (extremities) get the blur shadow — the newest positions are sharp.

### Analog + D-pad capture

The arcade stick captures both analog axes (axes 0, 1) and D-pad buttons (12-15) simultaneously. If any D-pad direction is pressed, it overrides the analog position for that axis. This allows fightsticks that report either analog or digital input to work seamlessly.

---

## Light vs Dark Themes

### Dark theme (default)

```css
:root {
    --accent: #FFFFFF;
    --color-bg: #000000;
}
```

### Light theme

```css
:root {
    --accent: #000000;
    --color-bg: #FFFFFF;
}
```

### Adaptive theme (follows system preference)

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
}
```

> OBS Browser Source inherits the OS color scheme. If your OS is in dark mode, the overlay is dark; in light mode, it's light.
