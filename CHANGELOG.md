# Changelog

All notable changes to **Bubo Controller** are documented in this file.

**Versioning scheme:** `v1.YYMM.XXX`
- `v1` — published (public project, Unlicense)
- `YYMM` — year + month of the build (zero-padded)
- `XXX` — sprint number within the build month (3 digits, resets to `001` on month rollover)

Format inspired by [Keep a Changelog](https://keepachangelog.com/). This is the technical record of what changed and why, at the engineering level.

---

## [v1.2611.003] — 2026-11-09

Background throttling fix — overlay no longer freezes when OBS loses window focus.

### Problem

The overlay stopped reacting to controller input whenever the OBS window lost foreground focus on Windows (e.g., clicking a game on another monitor, Alt+Tab to another app). Clicking the OBS window reactivated it. Root cause: OBS's embedded Chromium (CEF) inherits Chrome's background throttling policy — it throttles `requestAnimationFrame` and freezes Gamepad API state when the window loses focus.

### Changed

- **WebHID adapter added (primary layer).** New `BuboHidAdapter` class reads input reports via `device.addEventListener('inputreport', cb)`. HID events are NOT affected by background throttling — the overlay continues reading the controller even when OBS has no focus. The adapter exposes an object compatible with the Gamepad API (`{ id, buttons:[{pressed,value}], axes:[...] }` — 16 buttons + 4 axes, Xbox standard), so `updateGamepadState`/`updateAnalogState` are unchanged.
- **WebHID permission flow.** `requestDevice()` requires a user gesture. A "Conectar controle (WebHID)" button covers the overlay, visible after the hint delay when no gamepad is detected. Auto-connect (`getDevices()`) handles previously-authorized devices without a prompt.
- **Xbox-standard report parser.** Report ID 0x03 (17 bytes): buttons bitmap, D-pad hat, LT/RT uint16, 4 axes int16. DualSense/Hori fight stick use different formats — for those, the Gamepad API fallback is more reliable.
- **`setInterval` polling replaces `requestAnimationFrame` for gamepad read.** `setInterval(16ms)` is less aggressively throttled than rAF in some CEFs. The `animate()` loop now only draws (triggers/trail); the `update()` poll lives in `setInterval`.
- **`visibilitychange`/`focus`/`pageshow` handlers** force an immediate re-poll when OBS regains focus — covers the gap between the focus event and the next setInterval tick.
- **Gamepad API is now the automatic fallback.** If WebHID is unavailable or permission is denied, `getGamepad()` falls back to `navigator.getGamepads()`. Still vulnerable to throttling, but the setInterval + focus handlers reduce the impact.

### Reverted

- **WebHID removal (v1.2610.009) reverted.** The "stability reasons" justification did not hold against the focus-freeze bug — WebHID was the solution, not the problem. README "What NOT to contribute" and "Architecture invariants" updated to reflect WebHID as the primary layer.

### Docs

- **README: new "Background throttling" section.** Explains the CEF throttling policy, the 3-layer fix (WebHID primary, setInterval paliative, Gamepad API fallback), how to verify WebHID is active via console, and known limitations (HTTPS/`file://` requirement, gesture requirement, Xbox-standard report format).
- **README: "Common CEF-specific issues"** gains an entry pointing to the new section.
- **ROADMAP: version drift fixed** (v1.2611.001 → v1.2611.003).

### Code hygiene

- **Comment archaeology sanitized.** 49 references to "Sprint NNN" / "v1.2610.00X" removed from `index.html` comments. History lives in CHANGELOG; code comments now describe the "what" and non-obvious "why" only. Zero behavioral change — JS validated via `node --check`.

---

## [v1.2611.002] — 2026-11-09

Agent profile sprint — docs only, zero runtime change.

### Changed

- **AGENT.md: universal profile v1.1 → v2.0.** The role is renamed from "Senior Project Manager" to **Senior Multifaceted Project Manager**: a single PM who simultaneously embodies the entire specialist team (a "one-man army"). The specialist perspectives (engineering, architecture, design, QA, strategy) are permanent facets of one agent — not alternating hats, not a convened council. Every substantive decision is audited by all relevant facets at once. Explicit hierarchy added: the maintainer sits at the top — product, brand, scope, and visibility decisions are the maintainer's; the agent owns technical delivery end-to-end, challenges ideas with arguments rather than agreeing by convenience, and defers to the maintainer's final word.

### Fixed

- **Compliance:** tag `v1.2611.001` was missing from the previous release sprint; created retroactively on its release commit to restore the tag sequence.

---

## [v1.2611.001] — 2026-11-05

Documentation overhaul sprint — docs only, zero runtime change (`index.html` touched only for source-comment sanitization).

### Changed

- **Documentation consolidated into the four essential documents** — README, CHANGELOG, ROADMAP, AGENT (was 15 markdown files). Everything a user, artist, or contributor needs now lives in README.md: the full customization reference (global colors, ABXY schemes, per-button colors, opacity system, labels, arcade trail), the controller-layout creation guide (SVG blueprint structure, naming conventions, workflow, anti-patterns), the label sprite pack recipe, the OBS testing matrix, contributing conventions, and the architecture invariants.
- **README rewritten against the actual runtime API** — all 7 URL parameters (including the `?gamepad=` index/id-substring/list modes), the real `:root` variables and defaults (background 95%, buttons outline-only in idle / 100% solid on press, knobs 50% → 100%, trail length 60 / radius 8 / blur 12px), the fixed Nox→Sol trail gradient, and both layouts documented on the 640×216 canvas. Fixes all stale references (768×324-era proportions tables, 20%/95% opacity defaults, missing `?gamepad=` row).
- **CHANGELOG rewritten as a pure technical record** — verbatim conversation excerpts, workspace paths, and internal cross-references removed; the v1.2610.009 entry completed to cover the full arcade-v2 series that had shipped without changelog coverage.
- **AGENT.md replaced by the universal agent operating profile (v1.0)** — identical across the maintainer's repositories, synced via the project hub. Carries no project content; this README's "Architecture invariants" section is the project-specific layer.
- **ROADMAP rewritten** — accurate current status (arcade v2 and gamepad selection marked shipped), lean structure.

### Removed

- `docs/AUDIT.md`, `docs/AUDIT_PLAN.md` — internal architecture audit history (the engineering record lives in CHANGELOG.md and git history).
- `README.pt.md` — English-only distribution (the PT-BR mirror had drifted and doubled per-sprint maintenance).
- `CUSTOMIZATION_GUIDE.md`, `DESIGN_GUIDE.md`, `VARIANTS.md`, `LABEL_PACKS_GUIDE.md`, `CONTRIBUTING.md`, `masks/README.md`, `masks/controllers/README.md`, `masks/labels/README.md` — absorbed into README.md (or retired where stale).

### Fixed

- Stale version references: README badge and ROADMAP build line updated to `v1.2611.001` (README badge was one sprint behind VERSION).
- `index.html` source comments sanitized (references to the internal project ecosystem and owner requests removed — comments only, zero behavior change).
- `.gitignore` comment block on maintainer-local files neutralized (ignore patterns kept).

---

## [v1.2610.009] — 2026-10-27

### Added

- **Arcade fightstick layout v2** — `masks/controllers/arcade.svg` redrawn to the 640×216 canvas (matching the Xbox layout, ending the letterboxing the legacy 768×324 arcade SVG had inside the 640×216 overlay). Thirds-plus-halves proportions, lowercase layer IDs, octagonal stick gate.
- **Stick trail palette Nox→Sol** — the arcade stick trail renders a 4-stop gradient via `--trail-color-1` … `--trail-color-4`: Nox `#8865F1` at the newest position → Sol `#F165A4` at the oldest, with intermediate stops interpolated linearly (`#AB65D9` at ⅓, `#CF65BD` at ⅔). The gradient is fixed — it does not follow `?color=` schemes.
- **Trail retuned** — trail length 60 frames (~1s at 60fps, was 30), trail radius 8 viewBox units (⅓ of the 24-unit knob, was 12), glow blur 12px. The knob renders white in front of the trail with a 3px neon glow ring.
- **Gamepad selection** (`?gamepad=`) for multi-controller setups:
  1. `?gamepad=N` — 0-based index. Unstable across boots (follows the OS USB enumeration order, which can be re-shuffled); out-of-range values fall back to index 0 with a log line, keeping the overlay alive when a pad is off.
  2. `?gamepad=<id substring>` — case-insensitive substring match against the pad's USB id string (e.g. `hori`, `0x0f0d`). Stable across boots — the Vendor/Product id does not change with ports or boot order.
  3. `?gamepad=list` — diagnostic mode: logs each connected pad (index → id) to the console whenever the set changes (never per-frame), so the exact id to match can be discovered once.
  - The active pad is logged once per selection change (`[bubo] gamepad ativo: <id>`).

### Fixed

- **Arcade joy visibility series** — trail color parsing hardened (`parseCssVarToRgb` with a hardcoded fallback, fixing a black-trail artifact when CSS variables resolve late); joy knob centered (compensates a −266px Y-offset from an outer wrapper when embedded); knob drawn in front of the trail; trail always in Nox/Sol even when `?color=xbox` is active.

---

## [v1.2610.008] — 2026-10-13

### Fixed

- **"No controller connected" hint repositioned to the bottom gap** — was positioned in the geometric center of the 216px-tall overlay, overlapping the D-pad. Now sits in the bottom 27px reserved zone: text in an 18px-tall band centered in the gap, leaving 4.5px above and 4.5px below (4.5 + 18 + 4.5 = 27). Two new CSS variables: `--hint-height: 18px`, `--hint-bottom-gap: 4.5px`. CSS-only change (no JS).

---

## [v1.2610.007] — 2026-10-13

### Changed

- **Xbox SVG: top-button positioning** — the 6 top buttons (VIEW, MENU, LB, LT, RB, RT) shifted +18 viewBox units down (Y translate `9` → `27`) for breathing room below the controller's top rounded border. ABXY, D-pad, and analog sticks unchanged.

### Added

- **VIEW/MENU icon rects re-embedded** — generic UI icons (two overlapping squares for View, hamburger for Menu) live in path-local coordinates inside the `view`/`menu` groups, so they follow the parent group transforms automatically on SVG refreshes. Still opt-in via `?viewmenu=1`.

---

## [v1.2610.006] — 2026-10-13

### Fixed

- **VIEW/MENU icons hidden by default** — icons now appear only with the new `?viewmenu=` URL parameter (accepts `1`/`true`/`yes`/`on`, case-insensitive). Without it, VIEW/MENU render as plain pill outlines.
- **Analog-stick boundary rings (JOY_REACH) visible again** — CSS specificity fix: the generic button-fill rule (0,0,3,1) was overriding the reach rule (0,0,2,2), inheriting `fill-opacity: 0`. Rewrote the reach rules with explicit ID selectors (`g#left_joy_reach > path`, `g#right_joy_reach > path`, `g#joy_reach > path` → specificity 0,1,1,2) plus `!important` on `fill-opacity: 1` and `fill-rule: evenodd`.
- **`?color=` ABXY colors visible in the outline again** — the v1.2610.005 outline CSS had hardcoded `var(--accent)` (always Nimbus). Reverted to `var(--button-color, var(--accent))`, so scheme colors apply to BOTH the idle outline and the pressed fill.

---

## [v1.2610.005] — 2026-10-13

### Added

- **New Xbox SVG model** — corrected L3/R3 knob at ⅔ of the reach radius (a cancelled previous attempt had it at ¼, arcade sizing). Background as a `<path id="BACKGROUND">` direct child of the root group; all buttons drawn as ring paths (outer + inner subpaths, `fill-rule: evenodd`); layer IDs in the lowercase convention.
- **`processSvg()` rewritten** — splits each button ring into two paths at runtime: a solid FILL (outer subpath, invisible in idle) inserted before the OUTLINE (ring, always visible). Skips the background, root group, and reach rings. Replaces the previous `replaceRingWithCircle`/`extractOuterSubpath` helpers.
- **New visual spec** — background Umbral at 95% opacity; buttons outline-only in idle (`--button-fill-opacity: 0`); pressed state fills with the outline or scheme color at 100% opacity; L3/R3 knobs Nimbus 50% idle → 100% pressed; overlay opacity 1.0.
- **Trigger wave masked to the trigger contour** — `.trigger-overlay { clip-path: inset(0 round var(--trigger-clip-radius)) }` with `--trigger-clip-radius: 9px` (viewBox units); the wave no longer leaks outside the trigger outline. Scales correctly with `?scale=`/`?fit=`.
- **Bilingual "no controller connected" hint** — shows after 5s without a gamepad (`--hint-delay-ms`), hides on connect, never reappears after the first connection (avoids mid-stream noise). PT-BR text when `<html lang="pt">`, English otherwise. System font stack at weight 900 (the intended Sofia Sans XC Black is not embedded — offline-first, no external fonts). Force-shown in debug mode.
- **L3/R3 label aliases** — the Xbox SVG uses `left_joy`/`right_joy` groups for the sticks; `collectElements()` now aliases `l3`/`r3` to them so the labels render at the knob centers and follow the knobs.
- **New CSS variables** — `--hint-color`, `--hint-opacity`, `--hint-font-weight`, `--hint-size`, `--hint-delay-ms`, `--hint-transition-ms`, `--trigger-clip-radius`.

---

## [v1.2610.004] — 2026-10-08

### Added

- **New Xbox SVG blueprint** — canvas 768×324 → **640×216** (exactly 1/6 width × 10% height of a 3840×2160 reference canvas; aspect ~2.37:1 → ~2.96:1). Root group `<g id="XBOX">`.
- **Semantic palette** — `--nimbus: #F2F2F2` (outlines, L3/R3 knobs) and `--umbral: #0D0D0D` (background, fills) as the source-of-truth variables; `--accent` and `--color-bg` now alias them (backwards compatible).
- **L3/R3 special base** — knob fill is Nimbus at 50% as the stick-click indicator, jumping to 95% when pressed; L3/R3 keep Nimbus even under `?color=` schemes (do not follow ABXY colors).

### Changed

- **Canvas migration** — `--overlay-width/height`, `OVERLAY_BASE_WIDTH/HEIGHT`, and the inline SVG template moved to 640×216; label sizes reduced proportionally (`--label-size` 22→18px, `--label-size-small` 16→13px).
- **Opacity defaults** — background 20%, button fill 95% (unpressed and pressed), knob 50% → 95%.
- **`view1` → `menu`** layer rename (export auto-suffix collision), keeping `BUTTON_MAP.menu = 9` working.

### Preserved

- LT/RT trigger wave effect (canvas-based pressure graph) unchanged. Arcade layout unchanged (pending its own 640×216 redraw, which landed in v1.2610.009).

---

## [v1.2610.003] — 2026-10-08

### Changed

- **Governance: `AGENT.md` kept public** — header framing added documenting that the operating manual is committed for governance transparency (invariants and review bar are auditable by anyone).

---

## [v1.2610.002] — 2026-10-08

### Fixed

- **Doc/code drift resolved** — `masks/README.md` and `DESIGN_GUIDE.md` rewritten for the inline-SVG architecture (both still described the removed PNG background, 8 and 2 stale references respectively); historical-note headers added to the internal audit docs; `README.pt.md` aligned with `README.md` (version badge, project tree).

---

## [v1.2610.001] — 2026-10-08

### Added

- **Governance baseline** — `AGENT.md` (Senior PM operating manual: versioning policy, documentation charter, git and secret discipline, architectural invariants, Definition of Done, Never-rules), `CHANGELOG.md`, `VERSION` file, version badge in `README.md`.

### Changed

- **Migration to the inline-SVG architecture** — `index.html` rewritten as a single-file overlay supporting `?controller=xbox|arcade`: controller SVGs inlined as `<template>` elements, cloned and animated by JS at runtime (~38 KB → ~71 KB). Button positions come from the SVG itself — no CSS-positioned HTML elements.

### Removed

- `masks/bg_dark.png` — the overlay draws from inline SVG; the PNG background (and its CSS crop) is gone.
