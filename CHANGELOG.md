# Changelog

All notable changes to **Bubo Controller** are documented in this file.

**Versioning scheme:** `v1.YYMM.XXX`
- `v1` — published (public project, Unlicense)
- `YYMM` — year + month of the build (zero-padded)
- `XXX` — sprint number within the build month (3 digits, zero-padded, resets to `001` on month rollover)

Format inspired by [Keep a Changelog](https://keepachangelog.com/). This is the technical record of what changed and why, at the engineering level.

---

## [v1.2610.002] — 2026-10-08


### Added
- **Track C doc cleanup** — resolved the doc/code drift logged in `v1.2610.001`:
  - `masks/README.md`: rewritten as a Track C index. The old file described `masks/bg_dark.png` (8 references) and the PNG-based architecture; the new file indexes `controllers/` and `labels/`, explains the Track C render pipeline (SVG inline as `<template>`, JS-cloned, CSS-animated), preserves a "Historical note (Track A/B → Track C)" section, and keeps a "Common mistakes (legacy)" subsection for anyone forking from an older version.
  - `DESIGN_GUIDE.md`: recontextualized for Track C. The "Base image system" section (3840×2160 PNG + crop) was replaced by a "Canvas system (SVG viewBox)" section (768×324 SVG). The "Central crop" section was replaced by "Overlay stage = controller canvas". The z-index hierarchy table was updated (`.controller-bg` PNG → SVG background `<rect>`). The "Planning a variant" checklist was rewritten for the SVG workflow (Inkscape → inline `<template>` → `CONTROLLERS` map). A "Track C note" callout was added to the header. Preserved all the still-valid proportions, spacing, and animation-timing tables (they describe the SVG's encoded values, not the removed PNG).
  - `docs/AUDIT.md`: added a "Nota histórica (v1.2610.002)" header explaining that the document records the Track A/B → Track C migration, and that references to `bg_dark.png` / `background-image` / `--crop-x` describe the **prior** state. Historical entries preserved verbatim (no rewriting of audit history).
  - `docs/AUDIT_PLAN.md`: added a matching historical header.
- `README.pt.md` aligned with `README.md`:
  - Added version badge (`v1.2610.001`) and a `**Versão:**` line explaining the `v1.YYMM.XXX` scheme.
  - Updated the "Estrutura do projeto" tree to include `AGENT.md`, `CHANGELOG.md`, `VERSION` (matching the English README).

### Changed
- `ROADMAP.md`: marked the "Doc/code consistency (Track C cleanup)" item as ✅ in the Current Status table and removed it from the Future Technical Improvements table (debt resolved this Sprint). Updated the version line to `v1.2610.002`. Added a new the decision log bullet for `v1.2610.002` recording the question about whether `AGENT.md` should be gitignored (decision pending maintainer — see Audit below).

### Audit
- `docs/AUDIT.md` received a Track C historical header (see Added). No new architectural decision entries this Sprint — this was a documentation-only Sprint.
- **Open question (pending maintainer decision):** whether `AGENT.md` should remain public or be moved to `.gitignore` and kept only in the chat environment. The agent (acting as Senior PM) analyzed the trade-offs and presented a recommendation; the final decision is deferred to the maintainer per `AGENT.md` §1.2 (governance changes require maintainer confirmation). No action taken on `AGENT.md` until the maintainer decides.

### Verification
- `grep -rin "bg_dark" .` now returns:
  - Historical references only in `docs/AUDIT.md`, `docs/AUDIT_PLAN.md` (now prefixed with historical-note headers).
  - The `AGENT.md` §9.4 reference to the drift (descriptive, kept as-is — describes the pre-v1.2610.002 state; will be revised in a future Sprint if the maintainer confirms the AGENT.md should stay public).
  - The `CHANGELOG.md` entries describing the removal (historical record, correct).
  - `PRIVATE_PLANS.md` (gitignored, not committed — out of scope).
- The two public docs that described the old PNG architecture as **current** (`masks/README.md`, `DESIGN_GUIDE.md`) no longer do — they now describe Track C as current and relegate PNG to "Legacy (Track A/B)".
- This was a documentation-only Sprint; no `index.html` change, so the OBS test matrix (AGENT.md §7.2) was not required. Agent Browser end-to-end verification (AGENT.md §7.3) was not required for the same reason.

### ⚠️ Compliance
- None. All 15 Never-Rules (AGENT.md §12) verified intact. `PRIVATE_PLANS.md` confirmed gitignored and not staged. No secrets in committed files.

### Sprint footer
Sprint: v1.2610.002

---

## [v1.2610.001] — 2026-10-08


### Added
- **`AGENT.md`** — Senior PM Operating Manual. Establishes: agent identity & authority (§1), project context (§2), the `v1.YYMM.XXX` versioning policy (§3), documentation charter with mandatory conversation-excerpt preservation (§4), git & secret discipline (§5), 8 architectural invariants (§6), SOTA Definition-of-Done checklist with mandatory Agent Browser verification (§7), Sprint workflow (§8), risk management (§9), communication & Relatório structure (§10), behavioral protocols (§11), 15 Never-Rules (§12), decision log policy (§13), and a change-history table for the document itself (§14).
- **`CHANGELOG.md`** — this file. First Sprint entry under the new versioning scheme.
- **`VERSION`** — root file containing `v1.2610.001` (single line).
- **Version badge** in `README.md` header (`v1.2610.001`).

### Changed
- **Repository sync (local → remote):** the remote `master` branch (single prior commit, "Initial commit") was updated to match the maintainer's local working tree, which is the more advanced version. Diff summary:
  - `index.html`: rewritten to **Track C** (SVG inline, JS-parsed). Single-file HTML now supports `?controller=xbox|arcade` via inlined `<template>` SVGs. File grew from ~38 KB to ~71 KB. Removed the `masks/bg_dark.png` background dependency.
  - `README.md`: updated "About" and "Features" to reflect multiple controller layouts (Xbox + Arcade) via swappable SVG blueprints. Updated URL-parameters table, documentation table, and project-structure tree. Added version badge and `AGENT.md`/`CHANGELOG.md` references.
  - `README.pt.md`: aligned with the English README's structural updates.
  - `CUSTOMIZATION_GUIDE.md`: updated for the Track C opacity-layering system and the arcade stick-trail tuning variables.
- **`ROADMAP.md`:** added a the decision log note and a new technical-improvement entry to address the documented doc-drift (see below).

### Removed
- `masks/bg_dark.png` — no longer used. The Track C `index.html` draws the controller from inline SVG, not from a PNG background. (The remote's "Initial commit" was the last version to need it.)

### Audit
- See `docs/AUDIT.md` for the historical Track A → Track C migration record.

### Known technical debt logged this Sprint
- **Doc/code drift (Track C consistency):** `masks/README.md` (8 references) and `DESIGN_GUIDE.md` (2 references) still describe the old PNG-based architecture (`bg_dark.png`), while `index.html` has fully migrated to Track C (SVG inline, no PNG). This drift pre-exists in the local version and was carried over faithfully during sync (no silent edits to maintainer content). Logged as a Sprint task in `ROADMAP.md` → "Future Technical Improvements" for resolution in an upcoming Sprint. Per `AGENT.md` §9.4, every Sprint touching `index.html` must re-verify these three docs for stale references.

### ⚠️ Compliance
None. All 15 Never-Rules (AGENT.md §12) verified intact.

### Verification
- Repository state confirmed: `git status` clean after staging; `git check-ignore -v PRIVATE_PLANS.md` returns `.gitignore:38:PRIVATE_PLANS.md`.
- File-level diff between local and remote verified before commit (see `Relatório`).

### Sprint footer
Sprint: v1.2610.001
