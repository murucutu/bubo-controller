# Roadmap

This document lists planned variants, future improvements, and how to contribute new variants. There's no commitment to timelines — it's a statement of intent and a panel of opportunities for anyone who wants to pick them up.

**Versioning:** `v1.YYMM.XXX` (see [AGENT.md](AGENT.md#3-versioning-policy-v1yymmxxx) and [CHANGELOG.md](CHANGELOG.md)). Current build: `v1.2610.004` (October 2026, Sprint 004).

---

## Table of Contents

- [Current Status](#current-status)
- [Planned Variants](#planned-variants)
- [Future Technical Improvements](#future-technical-improvements)
- [Contributing](#contributing)

---

## Current Status

| Component                          | Status | Notes                                                        |
| ---------------------------------- | ------ | ------------------------------------------------------------ |
| Xbox Series layout (640×216 canvas) | ✅ v1.2610.007 | New model with corrected L3/R3 knob (2/3 of reach). `masks/controllers/xbox.svg` (Affinity-exported, `viewBox="0 0 640 216"`, IDs lowercase). Inlined as `<template id="svg-xbox">`. Outline-only buttons (Nimbus) in idle; full-color fill (Nimbus or ABXY color) when pressed. Background Umbral at 95% opacity. JOY_REACH visible as ring around each knob. VIEW/MENU icons opt-in via `?viewmenu=1`. `?color=xbox` colors both outline AND pressed fill. Top buttons (VIEW/MENU/LB/LT/RB/RT) shifted +18 viewBox units down for breathing room from the top border. |
| Arcade fightstick layout           | 🟡 pending new SVG | Current `arcade.svg` still uses 768×324 (legacy). Maintainer will deliver the new 640×216 arcade SVG in a future Sprint. Until then `?controller=arcade` works but letterboxes inside the 640×216 overlay. |
| Overlay canvas 640×216 (~2.96:1)   | ✅ v1.2610.005 | `--overlay-width: 640px`, `--overlay-height: 216px`. OBS source recommendations updated. |
| Nimbus/Umbral default palette       | ✅ v1.2610.005 | `--nimbus: #F2F2F2` (outlines), `--umbral: #0D0D0D` (background + pressed fill). `--accent` and `--color-bg` alias these. |
| Per-button colors                   | ✅ v1.2610.006 | `--color-a/b/x/y/lb/rb/lt/rt/view/menu` and `--color-l3/r3` (mapped to `g#left_joy/right_joy`). Colors affect BOTH the outline AND the pressed fill (when `?color=xbox/playstation` is active). Default (no scheme): both are Nimbus. |
| L3/R3 special base (Nimbus 50% idle, 100% pressed) | ✅ v1.2610.005 | Knob fill is `--nimbus` at 50% opacity as the stick-click indicator; jumps to 100% (full opaque) when L3/R3 pressed. L3/R3 keep Nimbus even under `?color=xbox` (do not follow ABXY scheme). |
| Opacity system (bg 95% / button 0% idle / 100% pressed) | ✅ v1.2610.005 | `--bg-opacity: 0.95`, `--button-fill-opacity: 0` (idle outline-only), `--button-pressed-opacity: 1` (full solid). Overlay opacity = 1.0 (was 0.85). |
| LT/RT trigger wave effect (Xbox, masked) | ✅ v1.2610.005 | The `<canvas>`-based pressure wave is masked to the trigger contour via CSS `clip-path: inset(0 round 9px)` — no longer leaks outside the trigger outline. |
| Auto-scale                          | ✅      | JS reads viewport, applies `transform: scale(var(--scale))` |
| Manual scale override (URL)         | ✅      | `?scale=2.0` in URL                                          |
| Fit modes (URL)                     | ✅      | `?fit=contain` (default) \| `?fit=cover` (may clip LT/RT)   |
| Debug mode                          | ✅ v1.2610.005 | Double-click on the overlay. Also force-shows the "no controller connected" hint for visual QA. |
| Text labels (Phase 1)               | ✅ v1.2610.005 | `--label-*` in `:root`, `<text>` per button, pressed state with inverted color. L3/R3 labels now render at the knob centers (alias `l3` → `left_joy`, `r3` → `right_joy`). |
| Icon sets (Phase 2)                 | ✅      | `?icons=xbox\|playstation\|nintendo\|none` in URL            |
| Arcade stick ghost-glow trail        | ✅      | `--stick-trail-*` variables, 1/3 knob size, analog + D-pad capture |
| ABXY color schemes via URL           | ✅ v1.2610.005 | `?color=xbox\|playstation` — affects only the pressed fill, not the outline. |
| Sprite pack guide (Phase 5 docs)    | ✅ docs | [LABEL_PACKS_GUIDE.md](LABEL_PACKS_GUIDE.md) + `masks/labels/sprite_example.svg` |
| Sprite pack runtime (Phase 5 impl) | 🟡 planned | Model ready, `<use>` HTML integration pending              |
| Inline SVG icons for View/Menu (Phase 3) | ✅ v1.2610.005 | Generic UI metaphors (two-squares for View, hamburger for Menu) embedded as `<rect class="button-icon">` in the SVG template — invariant §6.4 (no brand art) preserved. |
| "No controller connected" hint        | ✅ v1.2610.008 | Bilingual (`<html lang="pt">` → "Conecte um controle", default → "Connect a controller"). Shows after 5s of no gamepad, hides when one connects, never reappears after first connection. **Positioned in the bottom 27px gap** (Y=189→216 of the 216px overlay) — text in an 18px-tall band centered with 4.5px above and below (4.5+18+4.5=27), no longer overlaps the controller body. System font weight 900 fallback for Sofia Sans XC Black (invariant §6.3 preserved). |
| Doc/code consistency (Track C cleanup) | ✅ resolved in v1.2610.002 | `masks/README.md` and `DESIGN_GUIDE.md` rewritten for Track C; `docs/AUDIT.md` and `docs/AUDIT_PLAN.md` received historical-note headers. |

---

## Planned Variants

The Xbox Series base is the **open core** of the project. The Arcade fightstick layout is now also part of the open core (shipped in `v1.2610.001` via `?controller=arcade`). Variants for other controllers are planned for the future. **Detailed plans (layouts, button mappings, architectural decisions) are kept private by the maintainer** during development — finished variants may or may not be open-sourced, depending on context.

If you want to contribute a variant (e.g., a different controller layout), feel free to open a PR. The architecture (single-file HTML, Gamepad API, CSS variables `--color-*`/`--label-*`) is documented in [DESIGN_GUIDE.md](DESIGN_GUIDE.md), [CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md), and [VARIANTS.md](VARIANTS.md).

### Shipped (open core)

- **Xbox Series layout** — `?controller=xbox` (default).
- **Arcade fightstick layout** — `?controller=arcade`, Viewlix layout, 6 action buttons + stick, ghost-glow trail. Shipped `v1.2610.001`.

### Variants under consideration

- **PlayStation DualSense** with ✕ ○ □ △ symbols, touchpad, and central PS button.
- **Nintendo Pro Controller** with swapped A/B/X/Y layout (A right, B bottom) and +/−/Home buttons.
- **Premium arcade variant** with original art.

Custom variants may be commissioned from the maintainer.

---

## Future Technical Improvements

These are infrastructure improvements that benefit any future variant:

| Improvement                           | Priority  | Difficulty | Notes                                                   |
| ------------------------------------- | --------- | ---------- | ------------------------------------------------------- |
| Multi-gamepad support                  | Medium    | Low        | `?gamepad=2` URL param selects index                   |
| Auto light/dark theme                  | Low       | Low        | `prefers-color-scheme` in CUSTOMIZATION_GUIDE.md       |
| `devicePixelRatio` on trigger canvas   | Low       | Low        | Crisp rendering on retina/4K                           |
| `?color=` URL override                 | Low       | Low        | Live `--accent` override (already partially working via `?color=` scheme) |
| `?opacity=` URL override               | Low       | Low        | Adjust opacity without editing CSS                       |
| Sofia Sans XC Black font embedding     | Low       | Low        | Embed the font file as base64 in CSS for exact visual match on the hint (currently system font weight 900 fallback). Maintainer to provide the .ttf in a future Sprint. |
| Visual automated tests (Playwright)   | Low       | High       | Verify layout, colors, reactions                        |
| Sprite pack runtime (Phase 5)          | Medium    | Medium     | See [LABEL_PACKS_GUIDE.md](LABEL_PACKS_GUIDE.md) — model ready, integration pending |

---


## Contributing

The project is **public domain** (see [LICENSE](LICENSE)). There's no formal contribution process. But if you want to add a variant or fix:

1. **Fork the repository** on GitHub (or download the ZIP and version locally).
2. Make your changes — follow [DESIGN_GUIDE.md](DESIGN_GUIDE.md) for visual variants.
3. **Test in OBS** with a real controller. Working in the browser isn't enough — it has to work in OBS's CEF.
4. Update relevant documentation:
   - New layout? Update `DESIGN_GUIDE.md` or create a `DESIGN_GUIDE_<LAYOUT>.md`.
   - New colors or themes? Update `CUSTOMIZATION_GUIDE.md`.
   - Technical change? Update `docs/AUDIT.md` with the corresponding entry.
5. **Open a PR** or publish on your own fork — both are welcome.

See [CONTRIBUTING.md](CONTRIBUTING.md) for code conventions and OBS testing tips.

### Don't contribute

- WebHID (removed — see [docs/AUDIT.md](docs/AUDIT.md)). Don't reintroduce it.
- Build pipelines that break the "single-file HTML" constraint. If you need a build, do it as a separate fork.
- External dependencies (npm packages, CDNs, etc.). The overlay must run offline.
