# Roadmap

This document lists planned variants, future improvements, and how to contribute new variants. There's no commitment to timelines — it's a statement of intent and a panel of opportunities for anyone who wants to pick them up.

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
| Xbox Series layout                 | ✅      | Implemented in `index.html`                                  |
| Overlay at controller's natural aspect (768×324, ~2.37:1) | ✅ | No transparent padding; matches `bg_dark.png` crop          |
| Per-button colors                   | ✅      | `--color-a`, `--color-b`, …, `--color-menu`                  |
| Auto-scale                          | ✅      | JS reads viewport, applies `transform: scale(var(--scale))` |
| Manual scale override (URL)         | ✅      | `?scale=2.0` in URL                                          |
| Fit modes (URL)                     | ✅      | `?fit=contain` (default) \| `?fit=cover` (may clip LT/RT)   |
| Black-and-white default theme       | ✅      | Default `--accent: #FFFFFF` over `masks/bg_dark.png`         |
| Debug mode                          | ✅      | Double-click on the overlay                                   |
| Text labels (Phase 1)               | ✅      | `--label-*` in `:root`, `::after` per button, pressed state with inverted color |
| Icon sets (Phase 2)                 | ✅      | `?icons=xbox\|playstation\|nintendo\|none` in URL            |
| Sprite pack guide (Phase 5 docs)    | ✅ docs | [LABEL_PACKS_GUIDE.md](LABEL_PACKS_GUIDE.md) + `masks/labels/sprite_example.svg` |
| Sprite pack runtime (Phase 5 impl) | 🟡 planned | Model ready, `<use>` HTML integration pending              |
| Inline SVG icons for View/Menu (Phase 3) | 🟡 planned | Inline SVG in HTML, replacing empty `--label-view`           |

---

## Planned Variants

The Xbox Series base is the **open core** of the project. Variants for other controllers are planned for the future. **Detailed plans (layouts, button mappings, architectural decisions) are kept private by the maintainer** during development — finished variants may or may not be open-sourced, depending on context.

If you want to contribute a variant (e.g., a different controller layout), feel free to open a PR. The architecture (single-file HTML, Gamepad API, CSS variables `--color-*`/`--label-*`) is documented in [DESIGN_GUIDE.md](DESIGN_GUIDE.md) and [CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md).

### Variants under consideration

- **Arcade fightstick** for fighting games. Layout with 6 action buttons + 1 stick. Art and mapping specifics being defined.
- **PlayStation DualSense** with ✕ ○ □ △ symbols, touchpad, and central PS button.
- **Nintendo Pro Controller** with swapped A/B/X/Y layout (A right, B bottom) and +/−/Home buttons.
- **Premium arcade variant** with original art (potentially monetizable — may stay closed-source in the maintainer's private repo).

> If you're a streamer/professional wanting a specific variant now, contact the maintainer. Custom variants are one of the project's monetization paths.

---

## Future Technical Improvements

These are infrastructure improvements that benefit any future variant:

| Improvement                           | Priority  | Difficulty | Notes                                                   |
| ------------------------------------- | --------- | ---------- | ------------------------------------------------------- |
| Multi-gamepad support                  | Medium    | Low        | `?gamepad=2` URL param selects index                   |
| Auto light/dark theme                  | Low       | Low        | `prefers-color-scheme` in CUSTOMIZATION_GUIDE.md       |
| `devicePixelRatio` on trigger canvas   | Low       | Low        | Crisp rendering on retina/4K                           |
| `?color=` URL override                 | Low       | Low        | Live `--accent` override                                |
| `?opacity=` URL override               | Low       | Low        | Adjust opacity without editing CSS                       |
| "No controller connected" state        | Medium    | Medium     | Subtle hint to plug in a controller                     |
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
