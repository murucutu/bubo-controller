# Changelog

All notable changes to **Bubo Controller** are documented in this file.

**Versioning scheme:** `v1.YYMM.XXX`
- `v1` — published (public project, Unlicense)
- `YYMM` — year + month of the build (zero-padded)
- `XXX` — sprint number within the build month (3 digits, zero-padded, resets to `001` on month rollover)

Format inspired by [Keep a Changelog](https://keepachangelog.com/). Conversation excerpts are quoted **verbatim** in their original language (PT-BR), per the documentation charter in [`AGENT.md`](AGENT.md#46-conversation-log-inclusion-rules-critical). Sensitive data (tokens, credentials) is redacted.

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
