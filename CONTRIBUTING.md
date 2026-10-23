# Contributing to Bubo Controller

First off: **thank you** for considering contributing. This is a public-domain project, and any contribution — bug report, fix, new variant, documentation improvement — is welcome.

There's no formal contribution process. Fork, modify, send a PR. The guidelines below make life easier for everyone.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Before contributing](#before-contributing)
- [How to contribute](#how-to-contribute)
- [Code conventions](#code-conventions)
- [Testing in OBS](#testing-in-obs)
- [What NOT to contribute](#what-not-to-contribute)

---

## Code of Conduct

Be kind. Be patient. Assume good faith. We're all here because we like gamepads and OBS overlays.

Harassment, discrimination, or toxic behavior won't be tolerated. If you experience any of this, contact the maintainer directly.

---

## Before contributing

- **Check existing issues/PRs** before opening a new one. Your bug/idea might already be in progress.
- **Test in OBS** with a real controller. The overlay working in a browser is necessary but not sufficient — OBS's CEF (Chromium Embedded Framework) has quirks.
- **Single-file HTML is intentional**. Don't refactor into multi-file builds unless you're doing it as a separate fork.

---

## How to contribute

### Reporting bugs

Open an issue with:

1. **OBS version** (Help → About)
2. **OS** (Windows 10/11, macOS, Linux distro)
3. **Controller** (model + connection: USB or Bluetooth)
4. **Browser Source URL** (including any `?` params)
5. **Expected behavior** vs **actual behavior**
6. **Screenshot or short video** if relevant
7. **Console errors** (right-click the Browser Source → Interact → right-click → Inspect, or set OBS to log browser console)

### Suggesting features

Open an issue with the `enhancement` label. Describe:
- What problem you're trying to solve (not just "add X")
- Why the current design doesn't solve it
- Your proposed solution (with examples)

### Submitting code

1. **Fork** the repo on GitHub.
2. Create a branch: `git checkout -b fix/short-description` or `feat/short-description`.
3. Make your changes following [Code conventions](#code-conventions).
4. **Test in OBS** (see [Testing in OBS](#testing-in-obs)).
5. Update documentation if relevant:
   - Visual change → update [CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md)
   - Architecture change → update [DESIGN_GUIDE.md](DESIGN_GUIDE.md)
   - Internal audit → add an entry to [docs/AUDIT.md](docs/AUDIT.md)
6. Commit with a clear message:
   - `feat: add ?gamepad= URL param for multi-controller selection`
   - `fix: trigger canvas not redrawing on color change`
   - `docs: translate CUSTOMIZATION_GUIDE to Spanish`
7. Open a PR. Reference any related issues (`Closes #42`).

---

## Code conventions

### HTML/CSS

- **Single-file**: keep everything in `index.html`. No external CSS files.
- **CSS variables for everything** a user might want to customize (colors, sizes, timing, opacity).
- **`var()` with fallback**: `color: var(--button-color, var(--accent))` so missing variables don't break the layout.
- **`color-mix()` OK**: supported in CEF 111+. Avoid `@layer` (still flaky in some CEF versions).
- **`aria-label`** on interactive elements, `aria-hidden` on decorative.

### JavaScript

- **Vanilla JS only**: no bundlers, no transpilers, no `npm install`.
- **IIFE wrapper**: `(() => { 'use strict'; ... })()` keeps the global scope clean.
- **Class-based architecture**: keep the `XboxControllerOverlay` (or future `ControllerOverlay`) class as the single entry point.
- **`requestAnimationFrame`** for the gamepad polling loop (already implemented).
- **`URLSearchParams`** for URL param parsing (already used).
- **No external dependencies**: the overlay must work offline, in a single HTML file, opened as `file://`.

### File naming

- Lowercase with hyphens for filenames: `LABEL_PACKS_GUIDE.md`, `sprite_example.svg`.
- PascalCase for CSS classes: `.controller-overlay`, `.analog-knob` (already followed).
- camelCase for JS variables and functions: `applyScale`, `readFitMode`.

### Comments

- Comments in English for new code (international contributors).
- Existing Portuguese comments in `index.html` are tolerated but new contributions should use English.
- **DO NOT** add `--` (double hyphen) sequences inside SVG XML comments — XML forbids it (see the lesson learned in `sprite_example.svg`'s rewrite history).

---

## Testing in OBS

The overlay working in a browser ≠ working in OBS. CEF has quirks. Always test in OBS before opening a PR.

### Minimum test matrix

1. **No controller connected**: open the overlay, confirm the controller drawing is centered, no errors in console.
2. **With controller**: connect a real gamepad (Xbox controller for the base layout, PlayStation for `?icons=playstation`, etc.), press each button, confirm the corresponding element lights up.
3. **Analog sticks**: move sticks to extremes, confirm the knob stays within the boundary (no overflow).
4. **Triggers**: press LT/RT partially (analog), confirm the fill grows proportionally.
5. **URL params**: test `?scale=2`, `?fit=cover`, `?icons=playstation` (separately and combined).
6. **Debug mode**: double-click the overlay, confirm guide borders appear.
7. **Reload**: in OBS, right-click the source → Refresh. Confirm the overlay reloads correctly.

### OBS console access

To inspect the browser console:
1. Right-click the Browser Source in OBS.
2. Click **Interact**.
3. Right-click inside the interaction window.
4. Click **Inspect Element** (or press F12).
5. Go to the **Console** tab to see errors and logs.

### Common CEF-specific issues

- **`color-mix()` not supported**: upgrade OBS. CEF 111+ required.
- **External `<use>` not loading**: ensure the SVG file is served from the same folder as `index.html` (use `file://` paths, not HTTP).
- **Gamepad not detected in OBS but works in browser**: check OS-level gamepad permissions; on Windows, ensure the controller is recognized by Game Controllers control panel.

---

## What NOT to contribute

- **WebHID**: removed for stability reasons (see [docs/AUDIT.md](docs/AUDIT.md)). The Gamepad API covers all realistic OBS overlay needs.
- **Build pipelines**: the single-file HTML constraint is intentional. If you need a build, do it as a separate fork.
- **External dependencies**: no npm packages, no CDNs, no fonts loaded from Google Fonts. The overlay must run offline.
- **Brand-protected art**: don't include Sony, Microsoft, Nintendo, or other brand logos in the overlay or `masks/`. Use generic shapes; the user can add their own branded art.
- **Premium variant plans**: if you want to develop the arcade fightstick variant specifically, contact the maintainer first. The arcade variant has a specific monetization strategy that may conflict with an open-source contribution.

---

## License

By contributing, you agree that your contributions will be licensed under [The Unlicense](LICENSE) (public domain). No attribution will be required for your contributions, and the maintainer may use them freely in the public-domain project or in any derivative works (including closed-source premium variants).
