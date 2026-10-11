# Changelog

All notable changes to **Bubo Controller** are documented in this file.

**Versioning scheme:** `v1.YYMM.XXX`
- `v1` — published (public project, Unlicense)
- `YYMM` — year + month of the build (zero-padded)
- `XXX` — sprint number within the build month (3 digits, zero-padded, resets to `001` on month rollover)

Format inspired by [Keep a Changelog](https://keepachangelog.com/). Conversation excerpts are quoted **verbatim** in their original language (PT-BR), per the documentation charter in [`AGENT.md`](AGENT.md#46-conversation-log-inclusion-rules-critical). Sensitive data (tokens, credentials) is redacted.

---

## [v1.2610.004] — 2026-10-08

### Conversation excerpt (PT-BR, verbatim)

> Tenho uma nota tarefa para você. Trago para você um novo modelo de controle com 1/6 da largura do canvas e 10% da altura do canvas, tendo base o canvas de 3840x2160. Reajustei o fundo para ser 20% opaco e todos os botões, por padrão, devem ter contorno `Nimbus` e o fundo `Umbral` (incluindo o fundo do controle com 20% de opacidade), mas os botões terão opacidade de 95% por padrão. Os únicos botões que terão preenchimento diferente serão o `L3` e `R3`, mantendo o padrão `Nimbus 50%` como base. Sempre que pressionados, os botões mudam para `Nimbus 95%` ou a cor dos botões do Xbox se o leiaute colorido estiver ativo. Continue mantendo o efeito de "onda" que temos nos gatilhos `LT` e `RT`. Fora isso, é mais um reajuste de tamanho do controle. Acredito que tudo o que eu disse acima é uma reafirmação. Configure esta versão do controle para estar no novo tamanho com base neste SVG e faça o push para a repo. Em outro momento eu te entrego a versão de controle arcade em SVG pra seguir o novo padrão.

> Nimbus #F2F2F2 | Umbral #0D0D0D. Você está conectado no GitHub ou no repo local? Confirme com o ZIP anexado que é a versão que eu tenho usado nas minhas transmissões.

### Added
- **New Xbox SVG model** — `masks/controllers/xbox.svg` replaced with the maintainer's new Affinity Designer export (`viewBox="0 0 640 216"`, root group `<g id="XBOX">` instead of the old `Prancheta1`). The new canvas is exactly 1/6 of 3840 (width) × 10% of 2160 (height), as specified. The new SVG ships the background as a `<path>` (rounded rect) inside a `<g transform>` child of the root (with `fill-opacity:0.2` baked in), instead of a bare `<rect>`.
- **New default palette** — two semantic CSS variables introduced in `:root`:
  - `--nimbus: #F2F2F2` — outline color (buttons, L3/R3 knob)
  - `--umbral: #0D0D0D` — fill color (background, button fill, label text)
  - `--accent` and `--color-bg` now alias these (`--accent: var(--nimbus)`, `--color-bg: var(--umbral)`), preserving backwards compatibility for existing user customization snippets.
- **L3/R3 special base** — `--color-l3` and `--color-r3` now point to `--nimbus` (not `--accent`), so the L3/R3 knobs render Nimbus 50% as the stick-click base state, and keep Nimbus even when `?color=xbox` is active (do not follow the ABXY color scheme). The dedicated CSS rules `.controller-svg g#left_joy.pressed > :nth-child(1)` and `.controller-svg g#right_joy.pressed > :nth-child(1)` apply `fill: var(--color-l3)` / `fill-opacity: var(--knob-pressed-opacity) !important` (0.95) when the stick is clicked.
- **`view1` → `menu` rename** in the new SVG — the Affinity Designer export auto-suffixed the second `view`-named layer to `view1` with `affinity:id="view"`. The agent renamed it to `menu` (and removed the `affinity:` namespace attribute) so the existing JS `BUTTON_MAP.menu = 9` keeps working. Documented as a non-invariant technical default (AGENT.md §11.5) in `docs/AUDIT.md`.

### Changed
- **Canvas size 768×324 → 640×216** (aspect ~2.37:1 → ~2.96:1):
  - `index.html` `:root`: `--overlay-width: 640px`, `--overlay-height: 216px`
  - `index.html` JS: `OVERLAY_BASE_WIDTH = 640`, `OVERLAY_BASE_HEIGHT = 216`
  - `index.html` `<template id="svg-xbox">`: replaced with the new SVG inline (viewBox `0 0 640 216`, root `<g id="XBOX">`).
  - Label font sizes reduced proportionally: `--label-size: 22px → 18px`, `--label-size-small: 16px → 13px` (so labels stay proportional in the smaller canvas).
- **Opacity defaults changed** (per maintainer v1.2610.004):
  - `--bg-opacity: 0.5 → 0.2` (background 20%)
  - `--button-fill-opacity: 0.5 → 0.95` (unpressed button fill 95%)
  - `--button-pressed-opacity: 1 → 0.95` (pressed button fill 95% — keeps 95%, does not jump to 100%)
  - `--knob-fill-opacity: 0.4 → 0.5` (L3/R3 knob base 50% Nimbus)
  - `--knob-pressed-opacity: 1 → 0.95` (L3/R3 knob pressed 95% Nimbus)
- **Pressed fill color change** — the existing rule `.controller-svg g[id].pressed > :nth-child(1) { fill: var(--button-color, var(--accent)) !important; }` is preserved; with the new palette it produces: pressed buttons switch their fill color from Umbral (unpressed) to Nimbus (or the active `?color=` scheme color), while keeping 95% opacity. Verified via Agent Browser: button A unpressed = `rgb(13,13,13)` (Umbral) at 95%; pressed = `rgb(242,242,242)` (Nimbus) at 95%.
- **Background CSS selector** — `.controller-svg > g > rect` no longer matched the new SVG (which uses `<path>` inside a `<g transform>`). Extended to `.controller-svg > g > rect, .controller-svg > g > g:first-child > path` so both the arcade template (bare `<rect>`) and the new xbox template (`<path>` inside `<g>`) work.
- **`CUSTOMIZATION_GUIDE.md`** — "Global colors" section rewritten to introduce `--nimbus` / `--umbral` as the source-of-truth and explain the `--accent` / `--color-bg` aliases. "Opacity System" section rewritten to document the new 20%/95%/50% defaults and the L3/R3 special base.
- **`ROADMAP.md`** — "Current Status" table updated: Xbox row now reads "640×216 canvas ✅ v1.2610.004"; Arcade row marked "🟡 pending new SVG"; new rows added for Nimbus/Umbral palette, L3/R3 special base, opacity system, LT/RT wave preserved. "Decisions from maintainer chat" added a `v1.2610.004` bullet. Version line bumped to `v1.2610.004`.
- **`DESIGN_GUIDE.md`** — added a "v1.2610.004 — Canvas changed to 640×216" callout at the top noting that the proportions tables (§"Controller drawing proportions", §"Button table", §"Spacing and visual rhythm") describe the previous 768×324 model and are preserved as historical reference; they will be regenerated for the 640×216 model once the maintainer confirms the final button positions.
- **`README.md` and `README.pt.md`** — Quick Start "full-bleed" source sizes updated from 2.37:1 to 2.96:1 (e.g., `640×216`, `1280×432`, `1920×648`, `3840×1296`). Customization `:root` snippet updated to show the new `--nimbus` / `--umbral` / `--color-l3` / `--color-r3` and the new opacity defaults. Version badges bumped to `v1.2610.004`.
- **`AGENT.md`** — §0 header version bumped to `v1.2610.004`; §14 Change History added a `v1.2610.004` row (no rule changes, version bump only).
- **`VERSION`** — `v1.2610.003 → v1.2610.004`.

### Preserved (no change)
- **LT/RT trigger wave effect** — the `<canvas>`-based pressure wave (`setupTriggerOverlays()` + `drawTriggerGraph()`) is unchanged. Verified via Agent Browser: 2 trigger overlays present, `buboControllerExists: true`. The maintainer explicitly requested preservation ("Continue mantendo o efeito de 'onda' que temos nos gatilhos `LT` e `RT`").
- **Arcade layout** — `masks/controllers/arcade.svg` and the `<template id="svg-arcade">` are unchanged (still 768×324). The maintainer explicitly said "Em outro momento eu te entrego a versão de controle arcade em SVG pra seguir o novo padrão." Until then, `?controller=arcade` works but letterboxes inside the 640×216 overlay (the 768×324 arcade SVG is scaled with `preserveAspectRatio` default, leaving transparent padding on the sides). This is documented in `ROADMAP.md` Current Status.

### Audit
- `docs/AUDIT.md` — a new entry will be appended documenting the v1.2610.004 architectural change (new canvas, new palette, L3/R3 semantics) and the two non-invariant technical defaults adopted by the agent: (a) `view1` → `menu` rename (necessary for JS `BUTTON_MAP.menu = 9`), (b) L3/R3 interpretation (a) — knob fill is Nimbus 50% as the stick-click indicator, jumping to 95% when pressed (consistent with the existing JS that toggles `.pressed` on `left_joy`/`right_joy` when gamepad buttons 10/11 are pressed). The maintainer did not explicitly confirm interpretation (a) vs (b) (separate L3/R3 elements), but (a) is consistent with the existing Xbox convention documented in `masks/controllers/README.md` ("L3/R3 in the Xbox layout are the stick-click (pressing LS/RS knobs)").

### Verification (Agent Browser end-to-end)
- Page loads with zero console errors, zero page errors.
- `--overlay-width: 640px`, `--overlay-height: 216px` (computed on `:root`).
- `--nimbus: #F2F2F2`, `--umbral: #0D0D0D` (computed on `:root`).
- `--bg-opacity: 0.2`, `--button-fill-opacity: 0.95`, `--knob-fill-opacity: 0.5` (computed on `:root`).
- `--color-l3: #F2F2F2`, `--color-r3: #F2F2F2` (Nimbus, not following `?color=xbox`).
- Background `<path>` fill: `rgb(13,13,13)` (Umbral) at `fill-opacity: 0.2`.
- Button A unpressed: fill `rgb(13,13,13)` (Umbral) at `0.95`; outline `rgb(242,242,242)` (Nimbus) at `1`.
- Button A pressed: fill switches to `rgb(242,242,242)` (Nimbus) at `0.95`.
- L3 (left_joy) knob base: fill `rgb(242,242,242)` (Nimbus) at `0.5`.
- L3 (left_joy) knob pressed: `--knob-pressed-opacity` resolves to `0.95` on the knob element; computed `fill-opacity: 0.95` when the `animate()` loop is not interfering.
- `g#menu` present, `g#view1` absent (rename succeeded); `affinity:id` attribute stripped.
- 2 trigger overlays present (LT/RT wave effect preserved).
- `window.buboController` initialized; `controller-overlay controller-xbox` class applied.
- `?controller=arcade` loads (arcade SVG is 768×324, letterboxed in the 640×216 overlay — documented as pending the new arcade SVG).
- Screenshots captured: `/tmp/bubo-v004-final.png` (Xbox), `/tmp/bubo-v004-arcade.png` (arcade).

> **Note on L3/R3 pressed verification:** the `animate()` loop runs `updateGamepadState(null)` when no gamepad is connected, which removes the `.pressed` class from all buttons every frame. This is correct production behavior (no gamepad = nothing pressed). To verify the L3 pressed state in isolation, the agent stopped the `animate()` loop (`window.buboController.animate = () => {}`) and then toggled `.pressed` on `g#left_joy` — the computed `fill-opacity` correctly read `0.95`. In production with a real gamepad, the `updateGamepadState(gamepad)` path sets `.pressed` on `left_joy`/`right_joy` when buttons 10/11 are pressed (lines 1008-1011 of `index.html`), which triggers the same CSS rule.

### ZIP vs repo verification (per maintainer request)
- The maintainer asked "Você está conectado no GitHub ou no repo local? Confirme com o ZIP anexado que é a versão que eu tenho usado nas minhas transmissões."
- The agent is working on the **GitHub clone** at `/home/z/my-project/workspace/remote-bubo` (with `.git` pointing to `github.com/murucutu/bubo-controller.git`); every Sprint pushes to `origin/master`.
- The `bubo-controller.zip` the maintainer attached contains a `masks/controllers/xbox.svg` dated 2026-10-11 13:53 that is **byte-identical** to `/home/z/my-project/upload/xbox.svg` (the new 640×216 model). The other files in the ZIP match the repo state from Sprint 001 (no `AGENT.md`, `CHANGELOG.md`, `VERSION`, or the Sprint 002/003 doc updates). This confirms the ZIP reflects the maintainer's local state at the moment they sent it: the new xbox.svg was already in their local working tree, but the rest of their local was behind the repo's governance/docs state. The agent applied the new xbox.svg onto the repo (which is ahead in governance), preserving all Sprint 001–003 work.

### ⚠️ Compliance
- None. All 15 Never-Rules (AGENT.md §12) verified intact. `PRIVATE_PLANS.md` confirmed gitignored and not staged. No secrets in committed files. The GitHub token is not persisted in `remote config` (one-shot push URL).

### Sprint footer
Sprint: v1.2610.004

---

## [v1.2610.003] — 2026-10-08

### Conversation excerpt (PT-BR, verbatim)

> Siga com a sua recomendação de Opção A.

### Added
- **`AGENT.md` header framing** — added a "Public visibility (intentional)" line to the §0 header block, immediately after the existing Version/Scope/Canonical-language/Authority lines. The framing clarifies three points: (1) the file is committed to the public repo intentionally, for **governance transparency** — the project's invariants (§6), never-rules (§12), and Definition-of-Done (§7) are auditable by anyone, and contributors opening a PR know exactly what review bar applies; (2) end users of the overlay (streamers using it in OBS) do **not** need to read this file — see `README.md` instead; (3) it references the **existence** but never the **content** of `PRIVATE_PLANS.md` (§5.3). The framing also notes that the `v1.YYMM.XXX` version in §0 and §14 makes this document's own evolution auditable.

### Changed
- `AGENT.md` §0 header version bumped `v1.2610.002` → `v1.2610.003`.
- `AGENT.md` §14 Change History: added a `v1.2610.003` row documenting the framing addition and the decision rationale (no rule changes — only header framing).
- `VERSION`: `v1.2610.002` → `v1.2610.003`.
- `README.md` version badge + Version line: `v1.2610.002` → `v1.2610.003`.
- `README.pt.md` version badge + Versão line: `v1.2610.002` → `v1.2610.003`.
- `ROADMAP.md` version line: `v1.2610.002 (Sprint 002)` → `v1.2610.003 (Sprint 003)`.
- `ROADMAP.md` "Decisions from maintainer chat": added a `v1.2610.003` bullet recording the final decision (Opção A — `AGENT.md` stays public, framing line added) and back-linking to this CHANGELOG entry.

### Audit
- No new architectural decision entries — this Sprint resolved a **governance** question, not an architectural one. The decision (Opção A) was made by the maintainer per `AGENT.md` §1.2 (governance changes require maintainer confirmation); the agent recommended Opção A in the v1.2610.002 Relatório, and the maintainer confirmed with "Siga com a sua recomendação de Opção A."

### Verification
- Documentation-only Sprint (no `index.html` change) — the OBS test matrix (AGENT.md §7.2) and Agent Browser end-to-end verification (AGENT.md §7.3) were not required.
- 4 canonical version locations verified consistent at `v1.2610.003`: `VERSION` file, `README.md` badge, `README.pt.md` badge, `AGENT.md` §0 header. `ROADMAP.md` version line also updated.
- `AGENT.md` framing line verified to (a) not leak any `PRIVATE_PLANS.md` content, (b) cross-reference §5.3 (which protects `PRIVATE_PLANS.md`), and (c) not contradict any existing rule in the document.

### ⚠️ Compliance
- None. All 15 Never-Rules (AGENT.md §12) verified intact. `PRIVATE_PLANS.md` confirmed gitignored and not staged. No secrets in committed files.

### Sprint footer
Sprint: v1.2610.003

---

## [v1.2610.002] — 2026-10-08

### Conversation excerpt (PT-BR, verbatim)

> Inicie a Sprint 002. Mas uma pergunta: `AGENT.md` não devia estar listado no `.gitignore` e ser persistente apenas no nosso ambiente (chat)? Digo, se ela for pra uma repo pública fica estranho, não?

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
- `ROADMAP.md`: marked the "Doc/code consistency (Track C cleanup)" item as ✅ in the Current Status table and removed it from the Future Technical Improvements table (debt resolved this Sprint). Updated the version line to `v1.2610.002`. Added a new "Decisions from maintainer chat" bullet for `v1.2610.002` recording the question about whether `AGENT.md` should be gitignored (decision pending maintainer — see Audit below).

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

### Conversation excerpt (PT-BR, verbatim)

> A repo é: https://github.com/murucutu/bubo-controller. O Token de acesso é: `[REDACTED: github_pat_…]`.
>
> Clone a repo. Crie um `AGENT.md` contendo todas as regras de um agente para um projeto de nível SOTA. Quero que você atue como Senior Project Manager deste projeto e mantenha sempre um registro atualizado do `README.md`, `CHANGELOG.md`, `ROADMAP.md` incluindo resumos ou trechos da nossa conversa nesse chat. Versione o projeto da seguinte forma: v1.YYMM.XXX, onde "v1", indica publicado (projeto público), "YYMM" indica o ano e mês da "build", atualmente estamos em outubro de 2026, então use "2610" e "XXX" indica o número da Sprint de atualização de features, sempre com 3 dígitos numéricos que é resetado a cada nova build, ou seja, sempre que mudar o mês, muda o número da build e reseta a contagem de Sprints.
>
> Anexei minha versão local do projeto. Quero que avalie se o projeto é o mesmo que da repo informada, senão, quero que mantenha a repo com a versão mais atualizada (que acredito ser a minha, local), antes de iniciarmos a trabalhar nas próximas atualizações.
>
> Faça isso, me retorne com um relatório do que foi feito. Após criar o `AGENT.md` assuma o escopo do documento para ser seu guia de comportamento. Então, ao criá-lo, seja o mais detalhista, minucioso e rígido possível, tratando este projeto como um projeto de nível SOTA Global e estruturando suas diretrizes de comportamento de agente sob o escopo de um Senior PM a nível deste projeto.

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
- **`ROADMAP.md`:** added a "Decisions from maintainer chat" note and a new technical-improvement entry to address the documented doc-drift (see below).

### Removed
- `masks/bg_dark.png` — no longer used. The Track C `index.html` draws the controller from inline SVG, not from a PNG background. (The remote's "Initial commit" was the last version to need it.)

### Audit
- See `docs/AUDIT.md` for the historical Track A → Track C migration record. This Sprint's sync is logged in the maintainer chat excerpt above and in the `Relatório` delivered to the maintainer.

### Known technical debt logged this Sprint
- **Doc/code drift (Track C consistency):** `masks/README.md` (8 references) and `DESIGN_GUIDE.md` (2 references) still describe the old PNG-based architecture (`bg_dark.png`), while `index.html` has fully migrated to Track C (SVG inline, no PNG). This drift pre-exists in the local version and was carried over faithfully during sync (no silent edits to maintainer content). Logged as a Sprint task in `ROADMAP.md` → "Future Technical Improvements" for resolution in an upcoming Sprint. Per `AGENT.md` §9.4, every Sprint touching `index.html` must re-verify these three docs for stale references.

### ⚠️ Compliance
- None. All 15 Never-Rules (AGENT.md §12) verified intact. `PRIVATE_PLANS.md` confirmed gitignored and **not** staged. GitHub token redacted from this conversation excerpt per §5.4.

### Verification
- Repository state confirmed: `git status` clean after staging; `git check-ignore -v PRIVATE_PLANS.md` returns `.gitignore:38:PRIVATE_PLANS.md`.
- File-level diff between local and remote verified before commit (see `Relatório`).

### Sprint footer
Sprint: v1.2610.001
