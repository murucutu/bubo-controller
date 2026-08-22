# `masks/` — Custom backgrounds

This folder contains the overlay's background images. The project ships with a single default image:

```
masks/
└── bg_dark.png     (3840×2160, dark/black background)
```

You can swap, create variations, or develop a background entirely from scratch. This guide explains how.

---

## Table of Contents

- [The default image](#the-default-image)
- [Technical specifications](#technical-specifications)
- [Reference grid (3840×2160)](#reference-grid-38402160)
- [How to create a background from scratch](#how-to-create-a-background-from-scratch)
- [How to swap the background in CSS](#how-to-swap-the-background-in-css)
- [Creating a light theme](#creating-a-light-theme)
- [Common mistakes](#common-mistakes)

---

## The default image

`bg_dark.png` is a **3840×2160 (4K UHD, 16:9)** image with the Xbox controller silhouette/artwork. It serves as:

1. **Visual background** — the drawn controller appears as the overlay's background art.
2. **Proportion reference** — all CSS dimensions derive from percentages of this image.

It's drawn so that the **lower vertical center** of the image (80%-95% height, 40%-60% width region) contains the controller artwork. CSS crops this region and uses it as the `.controller-bg` background.

> The image is dark/black by default because it matches most streams (streams usually have a dark background). But you can create a light version.

---

## Technical specifications

- **Format**: PNG (with or without alpha).
- **Dimensions**: 3840×2160 (4K UHD). It's the base of everything.
- **Recommended resolution**: 1× (3840×2160). No benefit in 2× (7680×4320) — the final overlay scales to the source size in OBS.
- **Color depth**: 8-bit RGBA (PNG-32). PNG-24 (RGB) also works if the background is fully opaque.
- **File size**: ideally < 500 KB. Use optimized compression (`pngquant`, `oxipng`).

> The default image is ~33 KB. Sizes above 1 MB will noticeably slow OBS loading.

---

## Reference grid (3840×2160)

Use this grid as a guide when drawing the controller art on the image. The percentages are what CSS expects.

```
0%──────────────────────────────────────100%
 │              [3840 px width]              │
0%──────────────────────────────────────100%
 │                                           │
 │           (empty space / dark)            │
 │                                           │
 │                ┌────────┐ ← y=80%         │
40%──────────►   │        │   (1728px)       │
 │                │ CONTROL │                 │
 │                │  ART    │                 │
 │                │ 768×324 │                 │
 │                └────────┘ ← y=95%         │
 │              (2052px)                     │
 │                                           │
 │           (empty space / dark)            │
 │                                           │
100%──────────────────────────────────────100%
                ▲
        x=40% (1536px)
                x=60% (2304px) ─►
```

### Key coordinates

| Mark                          | Pixels         | Percentage            |
| ------------------------------ | -------------- | --------------------- |
| Controller horizontal start    | x = 1536       | 40% of width          |
| Controller horizontal end      | x = 2304       | 60% of width          |
| Controller vertical start      | y = 1728       | 80% of height         |
| Controller vertical end        | y = 2052       | 95% of height         |
| Controller width               | 768px          | 20% of width          |
| Controller height              | 324px          | 15% of height         |
| Top row (LT/RT)               | 108px tall     | 5% of height          |
| Bottom row (rest)             | 216px tall     | 10% of height         |

### Controller center

```
Horizontal center: (1536 + 2304) / 2 = 1920 (image center)
Vertical center:   (1728 + 2052) / 2 = 1890
```

### Button positions on the base image (to draw art)

Multiply CSS coordinates by 5 (because CSS uses 768×324 and the image is 3840×2160, factor 5×). Or use percentages.

> For exact values, open `bg_dark.png` in an image editor and overlay a 5×5% grid with guides. See the grid above for zones.

---

## How to create a background from scratch

### Step 1 — Empty template

Create a 3840×2160 image with a solid black background (or transparent, but black matches streams better).

### Step 2 — Mark the crop

Draw a temporary guide rectangle (will be removed) at:

```
x: 1536, y: 1728, width: 768, height: 324
```

The controller artwork goes inside this rectangle.

### Step 3 — Draw the buttons

Inside the guide rectangle, draw each button at the position and size shown in the table in [DESIGN_GUIDE.md](../DESIGN_GUIDE.md#button-table-dimensions-and-positions). Values in DESIGN_GUIDE are in controller pixels (768×324 scale); multiply by 5 to get base image pixels.

### Step 4 — Art style

You can draw buttons in several ways:

- **Hollow silhouettes** (just outlines, transparent inside) — matches the current overlay which draws outlines via CSS.
- **Filled silhouettes** — appears below CSS outlines. Use if you want to give volume to the art.
- **Internal details** (textures, A/B/X/Y icons) — appear under the CSS outline.

### Step 5 — Remove guides

Erase the temporary guide rectangle and any other alignment marks.

### Step 6 — Export

Export as PNG, ideally with alpha. Compress with `pngquant` or `oxipng`:

```bash
oxipng -o 4 bg_dark.png
# or
pngquant --quality 60-80 --output bg_dark.png --force bg_dark.png
```

### Step 7 — Replace

Replace `masks/bg_dark.png` (or create a new file and update the CSS in `index.html`).

---

## How to swap the background in CSS

### Simple swap (same filename)

Just replace `masks/bg_dark.png` with another file of the same name. CSS already points to it.

### Swap to a different filename

Edit `background-image` in CSS:

```css
.controller-bg {
    background-image: url('masks/bg_custom.png');
    /* ... */
}
```

### Different crop (visible region different from image)

If your art is in a different position on the base image, adjust the crop variables in `:root`:

```css
:root {
    --base-width: 3840px;
    --base-height: 2160px;
    --crop-x: 1536px;     /* x where the controller starts on the base image */
    --crop-y: 1728px;     /* y where the controller starts on the base image */
}
```

### Different base image size

If your image is 1920×1080 (Full HD, instead of 4K):

```css
:root {
    --base-width: 1920px;
    --base-height: 1080px;
    --crop-x: 768px;      /* half of original 1536px */
    --crop-y: 864px;      /* half of original 1728px */
    --controller-width: 768px;  /* if keeping controller-stage at 768×324 */
    --controller-height: 324px;
}
```

> Shrinking the base image doesn't shrink the controller-stage. It only makes `background-size: var(--base-width) var(--base-height)` scale the image inside the stage. To change the stage size, also adjust `--controller-width` and `--controller-height`.

---

## Creating a light theme

For a light theme:

1. Create `masks/bg_light.png` with a white background (instead of black).
2. The controller art should have dark outlines (black or dark gray) on a white background.
3. In CSS:

```css
:root {
    --accent: #000000;     /* swap fallback color to black */
}
.controller-bg {
    background-image: url('masks/bg_light.png');
}
```

To auto-switch based on system theme:

```css
:root {
    --accent: #FFFFFF;
}
@media (prefers-color-scheme: light) {
    :root { --accent: #000000; }
    .controller-bg { background-image: url('masks/bg_light.png'); }
}
```

---

## Common mistakes

### Buttons misaligned with the image art

**Cause**: the art on the base image isn't exactly where CSS expects the crop.

**Fix**: open the image in an editor, overlay a reference grid (40-60% width, 80-95% height), and check if the image's buttons are where they should be. Adjust the image OR adjust `--crop-x`/`--crop-y`.

### Image is pixelated

**Cause**: image too small being scaled up, OR wrong `background-size`.

**Fix**: keep the base image at 3840×2160. Check that `--base-width`/`--base-height` match the image's actual dimensions.

### Image doesn't appear

**Likely cause**: wrong file path.

**Fix**: the `url('masks/bg_dark.png')` path is relative to `index.html`. If you moved `index.html` to another folder, adjust the path. In OBS, use the "Local file" option to point to `index.html` — relative paths continue to work.

### Solid colored background bleeding around the controller

**Cause**: the base image has a background color (not transparent) outside the controller crop.

**Fix**: the overlay's `body` is transparent, so what appears outside `.controller-stage` is only what's on the image. If the image has a solid black background outside the controller, it'll show. Use an image with alpha transparent outside the controller, OR accept the background as part of the overlay.

> To match stream scenes, it's usually better to have an image with a **transparent** background outside the controller. That way the overlay "floats" over the scene without a black box behind it.
