# Roadmap

Public, high-level roadmap. There's no commitment to timelines — it's a statement of intent and a panel of opportunities for anyone who wants to pick them up.

**Versioning:** `v1.YYMM.XXX` (see [CHANGELOG.md](CHANGELOG.md)). Current build: `v1.2611.006` (November 2026, Sprint 006).

---

## Current status

| Area | Status | Notes |
| ---- | ------ | ----- |
| Xbox Series layout (640×216) | ✅ shipped | Outline-only idle buttons, full-color pressed fill, pressure wave on LT/RT (masked to the trigger contour), calibrated sticks with visible reach rings |
| Arcade fightstick layout v2 (640×216) | ✅ shipped | Thirds+halves proportions, octagonal gate, ghost-glow stick trail in fixed Nox→Sol gradient |
| Canvas 640×216 (~2.96:1) | ✅ shipped | Both layouts share the canvas; full-bleed source sizes documented in [README.md](README.md) |
| Per-button colors (`--color-*`) | ✅ shipped | Colors apply to outline AND pressed fill |
| ABXY color schemes (`?color=xbox\|playstation`) | ✅ shipped | Official Xbox / PlayStation palettes |
| Icon sets (`?icons=xbox\|playstation\|nintendo\|none`) | ✅ shipped | Text labels positioned by JS at button centers |
| VIEW/MENU icons (opt-in) | ✅ shipped | Generic UI metaphors, hidden by default, `?viewmenu=1` |
| Gamepad selection (`?gamepad=`) | ✅ shipped | Index / USB-id substring (stable) / `list` diagnostic |
| "No controller" hint | ✅ shipped | Bilingual, bottom 27px gap, never reappears after first connection |
| Auto-scale + fit modes | ✅ shipped | `?fit=contain\|cover`, `?scale=N` |
| Debug mode | ✅ shipped | Double-click the overlay |
| Label sprite pack model | ✅ docs + model | `masks/labels/sprite_example.svg` + recipe in README — runtime integration pending |

## Planned variants

- **PlayStation DualSense** — ✕ ○ □ △ symbols, touchpad, central PS button.
- **Nintendo Pro Controller** — swapped A/B/X/Y layout (A right, B bottom), +/−/Home buttons.

Want a specific variant now? Open an issue or a PR — the architecture (single-file HTML, Gamepad API, CSS-variable customization API) is documented in [README.md](README.md), including the full [layout creation guide](README.md#adding-a-new-controller-layout).

## Future technical improvements

| Improvement | Priority | Difficulty | Notes |
| ----------- | -------- | ---------- | ----- |
| Sprite pack runtime integration | Medium | Medium | `<symbol>` + `<use>` model ready (see README); URL param to pick packs |
| `devicePixelRatio` on trigger canvas | Low | Low | Crisp rendering on retina/4K |
| `?opacity=` URL override | Low | Low | Adjust opacity without editing CSS |
| Auto light/dark theme | Low | Low | `prefers-color-scheme` media query |
| Sofia Sans XC Black embedding | Low | Low | Base64-embed the font for an exact hint match (currently system font weight 900 fallback) |
| Visual automated tests (Playwright) | Low | High | Verify layout, colors, reactions |

---

## Contributing

The project is **public domain** (see [LICENSE](LICENSE)). There's no formal contribution process — fork it, modify it, open a PR. Follow the [code conventions and OBS testing guide](README.md#contributing). The [architecture invariants](README.md#architecture-invariants) are hard constraints — a change that violates them is a defect, even if it works.
