# Roadmap

This document lists planned variants, future improvements, and how to contribute new variants. There's no commitment to timelines — it's a statement of intent and a panel of opportunities for anyone who wants to pick them up.

**Versioning:** `v1.YYMM.XXX` (see [AGENT.md](AGENT.md#3-versioning-policy-v1yymmxxx) and [CHANGELOG.md](CHANGELOG.md)). Current build: `v1.2610.002` (October 2026, Sprint 002).

---

## Table of Contents

- [Current Status](#current-status)
- [Planned Variants](#planned-variants)
- [Future Technical Improvements](#future-technical-improvements)
- [Decisions from maintainer chat](#decisions-from-maintainer-chat)
- [Contributing](#contributing)

---

## Current Status

| Component                          | Status | Notes                                                        |
| ---------------------------------- | ------ | ------------------------------------------------------------ |
| Xbox Series layout                 | ✅      | Implemented in `index.html` (Track C, SVG inline)            |
| Arcade fightstick layout           | ✅      | Implemented in `index.html` via `?controller=arcade` (Track C) — `masks/controllers/arcade.svg` |
| Overlay at controller's natural aspect (768×324, ~2.37:1) | ✅ | No transparent padding; CSS-only background (Track C, no PNG) |
| Per-button colors                   | ✅      | `--color-a`, `--color-b`, …, `--color-menu`                  |
| Auto-scale                          | ✅      | JS reads viewport, applies `transform: scale(var(--scale))` |
| Manual scale override (URL)         | ✅      | `?scale=2.0` in URL                                          |
| Fit modes (URL)                     | ✅      | `?fit=contain` (default) \| `?fit=cover` (may clip LT/RT)   |
| Black-and-white default theme       | ✅      | Default `--accent: #FFFFFF` over CSS background              |
| Debug mode                          | ✅      | Double-click on the overlay                                   |
| Text labels (Phase 1)               | ✅      | `--label-*` in `:root`, `::after` per button, pressed state with inverted color |
| Icon sets (Phase 2)                 | ✅      | `?icons=xbox\|playstation\|nintendo\|none` in URL            |
| Arcade stick ghost-glow trail        | ✅      | `--stick-trail-*` variables, 1/3 knob size, analog + D-pad capture |
| ABXY color schemes via URL           | ✅      | `?color=xbox\|playstation`                                   |
| Opacity layering system              | ✅      | 40% bg + 40% buttons ≈ 64% combined; 100% on press          |
| Sprite pack guide (Phase 5 docs)    | ✅ docs | [LABEL_PACKS_GUIDE.md](LABEL_PACKS_GUIDE.md) + `masks/labels/sprite_example.svg` |
| Sprite pack runtime (Phase 5 impl) | 🟡 planned | Model ready, `<use>` HTML integration pending              |
| Inline SVG icons for View/Menu (Phase 3) | 🟡 planned | Inline SVG in HTML, replacing empty `--label-view`           |
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
- **Premium arcade variant** with original art (potentially monetizable — may stay closed-source in the maintainer's private repo). Community PRs touching the premium arcade scope require maintainer sign-off (see [CONTRIBUTING.md](CONTRIBUTING.md)).

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

## Decisions from maintainer chat

This section records roadmap-level decisions originating from maintainer chat conversations. Per-verbatim excerpts live in [CHANGELOG.md](CHANGELOG.md); this section holds the one-line decisions and back-links.

- **v1.2610.001** — Adopted the `v1.YYMM.XXX` versioning scheme (v1 = published, YYMM = build month, XXX = sprint number, resets monthly). Established the Senior PM operating manual ([AGENT.md](AGENT.md)) as the binding behavior guide for the agent. Synced the maintainer's local Track C version to the remote repo (the remote's "Initial commit" was the older PNG-based version). Back-link: [CHANGELOG.md v1.2610.001](CHANGELOG.md#v1261001--2026-10-08).
- **v1.2610.002** — Resolved the doc/code drift logged in v1.2610.001 (Track C doc cleanup in `masks/README.md`, `DESIGN_GUIDE.md`, `docs/AUDIT.md`, `docs/AUDIT_PLAN.md`; `README.pt.md` aligned with `README.md`). Maintainer raised an open question about whether `AGENT.md` should be gitignored and kept only in the chat environment; the agent analyzed trade-offs and presented a recommendation; the final decision is pending maintainer confirmation. Back-link: [CHANGELOG.md v1.2610.002](CHANGELOG.md#v1261002--2026-10-08).

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
