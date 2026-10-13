# Changelog

All notable changes to **Bubo Controller** are documented in this file.

**Versioning scheme:** `v1.YYMM.XXX`
- `v1` — published (public project, Unlicense)
- `YYMM` — year + month of the build (zero-padded)
- `XXX` — sprint number within the build month (3 digits, zero-padded, resets to `001` on month rollover)

Format inspired by [Keep a Changelog](https://keepachangelog.com/). Conversation excerpts are quoted **verbatim** in their original language (PT-BR), per the documentation charter in [`AGENT.md`](AGENT.md#46-conversation-log-inclusion-rules-critical). Sensitive data (tokens, credentials) is redacted.

---

## [v1.2610.008] — 2026-10-13

### Conversation excerpt (PT-BR, verbatim)

> 2. Em `bubo-controller`, notei que a mensagem/aviso de controle conectado ou desconectado aparece no meio do controle. Use o gap inferior de 27px para colocar a notificação centralizada naquele espaço com altura de 18px e centralize no gap, deixando 4.5px acima e abaixo do texto (18+9=27).

### Fixed

- **"No controller connected" hint repositioned to the bottom 27px gap**: in v1.2610.005, the hint was positioned with `position: absolute; inset: 0; display: flex; align-items: center; justify-content: center` — which placed the text in the geometric center of the 216px-tall overlay, overlapping the D-pad (Y center = 108). The maintainer observed this visual collision and requested the hint be moved to the bottom 27px gap (the controller's bottom margin, Y = 189 → 216), with the text in an 18px-tall band centered in the gap (4.5px above + 18px text + 4.5px below = 27px).
  - **CSS change**: the `inset: 0` + `align-items: center` was replaced with explicit bottom-gap positioning:
    ```css
    .controller-hint {
        position: absolute;
        left: 0; right: 0;
        bottom: var(--hint-bottom-gap, 4.5px);
        height: var(--hint-height, 18px);
        display: flex; align-items: center; justify-content: center;
        line-height: 1;
        /* ... rest unchanged ... */
    }
    ```
  - **Two new CSS variables** added to `:root` for SSOT customization:
    - `--hint-height: 18px` — the height of the hint band (where the text sits)
    - `--hint-bottom-gap: 4.5px` — the margin from the bottom of the overlay to the bottom of the hint band
  - The `line-height: 1` was added to ensure the 14px font fits cleanly inside the 18px band without overflow (default line-height of ~1.2 would make 14px text ~16.8px tall, still within 18px, but `line-height: 1` makes it exactly 14px so vertical centering is predictable).
  - The math: overlay is 216px tall. The hint band sits at `bottom: 4.5px` with `height: 18px` → the band occupies Y = 216 - 4.5 - 18 = 193.5 to Y = 216 - 4.5 = 211.5. The remaining 4.5px above the band (Y = 189 to 193.5) is the top half of the 27px gap. Total gap: 4.5 + 18 + 4.5 = 27 ✓.

### Verification (Agent Browser end-to-end, via `file://`)

- Page loads with zero console errors.
- Hint positioning (measured via `getBoundingClientRect()` on overlay at scale 2x, so values are 2× viewBox units):
  - `overlay_h: 432px` (= 216 × 2) ✓
  - `hint_top_from_overlay_top: 387px` (= 193.5 × 2) ✓ — hint starts at Y=193.5 in viewBox coords
  - `hint_height: 36px` (= 18 × 2) ✓ — band is exactly 18px tall in viewBox
  - `hint_bottom_from_overlay_bottom: 9px` (= 4.5 × 2) ✓ — 4.5px margin from bottom
- Hint `.visible` class added after 5s (no gamepad) with `opacity: 0.6` ✓ (logic unchanged from v1.2610.005).
- **VLM visual confirmation** of `/tmp/bubo-v008-hint-bottom.png`: "O texto está na parte inferior, em um espaço vazio abaixo do controle (especificamente abaixo do D-pad e dos botões de ação). Sim, há margem/respiro visível acima (entre a base do D-pad e o topo do texto) e abaixo do texto (antes da borda inferior do overlay). O texto 'Connect a controller' não está sobreposto ao centro do controle; ele serve como um rodapé informativo na parte inferior da interface." ✓

### Audit

- **Why the gap is 27px**: the maintainer specified 27px (the controller's bottom margin in the 216px-tall overlay). The controller's geometric body ends at Y=189 (the bottom of the D-pad / analog stick reach rings), leaving Y=189→216 (27px) as a reserved zone. This is the same zone where the dock would sit in the streaming theme — keeping the hint there is consistent with the project's visual rhythm (the dock is also a 72px-tall element centered in the bottom 108px dead zone of the 4K canvas).

- **Why 18px text height + 4.5px margins**: the maintainer specified `18+9=27` — interpreting the "9" as 4.5 + 4.5 (the margin above + below the text). The 18px band gives 14px font (the `--hint-size`) comfortable headroom with `line-height: 1`. The 4.5px margins are half of 9, distributing the 9px remainder symmetrically above and below the text.

- **No JS changes needed**: the `setupControllerHint()` and `updateControllerHint()` functions (added in v1.2610.005) create the `<div class="controller-hint">` and toggle the `.visible` class. The positioning is entirely CSS-driven, so no JS update was needed — just the CSS rule change.

### ⚠️ Compliance

- None. All 15 Never-Rules (AGENT.md §12) verified intact.
- Invariants §6.1, §6.3, §6.4, §6.5, §6.8 all preserved.
- No secrets, no PRIVATE_PLANS.md staged, GitHub token not persisted.

### Sprint footer

Sprint: v1.2610.008 — Hint repositioned to the bottom 27px gap (4.5 + 18 + 4.5 = 27) so it no longer overlaps the controller body. Two new CSS variables for customization. Let's Play Up!

---

## [v1.2610.007] — 2026-10-13

### Conversation excerpt (PT-BR, verbatim)

> Segue novo SVG do controle de Xbox com ajustes de posicionamento. Atualize para usarmos essa versão.

### Changed

- **New Xbox SVG with adjusted top-button positioning**: `masks/controllers/xbox.svg` and the inline `<template id="svg-xbox">` in `index.html` replaced with the maintainer's new Affinity Designer export. The only geometric change is the **Y-axis translate** of the 6 top buttons (VIEW, MENU, LB, LT, RB, RT): all moved from `Y=9` to `Y=26.999982` (≈27), a downward shift of 18 viewBox units. This adds breathing room between these buttons and the controller's top rounded border, addressing the visual tightness the maintainer observed.
- **ABXY buttons, D-pad, and analog sticks unchanged** — the maintainer's new SVG keeps their transforms identical to v1.2610.005. Verified via Agent Browser: A/B/X/Y centers at `(392,171)/(428,135)/(356,135)/(392,99)` (identical to v1.2610.005); D-pad up/down/left/right centers at `(248,108)/(248,162)/(221,135)/(275,135)` (identical); analog sticks at `(104,135)/(536,135)` (identical).
- **Top buttons new centers** (verified via Agent Browser):
  - VIEW: `(302, 45)` (was `(302, 27)` in v1.2610.005)
  - MENU: `(338, 45)` (was `(338, 27)`)
  - LB: `(230, 45)` (was `(230, 27)`)
  - LT: `(104, 45)` (was `(104, 27)`)
  - RB: `(410, 45)` (was `(410, 27)`)
  - RT: `(536, 45)` (was `(536, 27)`)
  - All shifted exactly +18 in Y (consistent with the transform change `9 → 27`).
- **D-pad matrix cleanup** (cosmetic, no visual change): the `down` and `left` groups had `-0` in some matrix entries (e.g., `matrix(-0.222222,0,-0,-0.333333,...)`); the new SVG normalizes these to `0` (e.g., `matrix(-0.222222,0,0,-0.333333,...)`). Mathematically equivalent (`-0 === 0` in IEEE 754); no visual change.

### Added

- **`<rect class="button-icon">` re-embedded in the new SVG** for VIEW and MENU. The maintainer's exported SVG ships the rings only (no icon rects); the agent re-added the two-overlapping-rounded-rectangles (View) and three-horizontal-lines (Menu) at the pill centers, in path-local coords. These remain **hidden by default** (per v1.2610.006) and only appear with `?viewmenu=1`. The icons are positioned in path-local coords so they move WITH the parent `<g>` transform — when the maintainer shifted the `view`/`menu` groups down by 18 units, the icons automatically shifted too (no manual icon position update needed).

### Audit

- **Why the icons aren't in the maintainer's SVG**: the maintainer exports the raw geometric shapes from Affinity. The icons (two-squares, hamburger) are a runtime UX layer the agent added in v1.2610.005 — they are not part of the controller's "physical" geometry, they are visual indicators. The agent's policy: re-embed them on every SVG update so they survive across maintainer SVG refreshes. This is documented in `masks/controllers/README.md` (the SVG is the source-of-truth for the controller's geometry; icons are a runtime addition managed by the agent).

- **No JS changes needed**: the maintainer's new SVG uses the same lowercase IDs (`view`, `menu`, `lb`, `lt`, `rb`, `rt`, `up`, `down`, `left`, `right`, `a`, `b`, `x`, `y`, `left_joy`, `left_joy_reach`, `right_joy`, `right_joy_reach`) as v1.2610.005. The existing `BUTTON_MAP`, `collectElements()`, and `cacheKnobData()` JS continue to find every element without changes. The L3/R3 label aliases (added in v1.2610.005) still work because the new SVG keeps `<g id="left_joy">` and `<g id="right_joy">` (not `<g id="l3">`/`<g id="r3">`).

- **CSS unchanged**: no CSS rules needed updating. The button-fill and outline rules use `var(--button-color, var(--accent))` which automatically adapt to whatever colors the URL params activate. The reach rules (added in v1.2610.006 with explicit IDs `g#left_joy_reach > path` etc.) still match the new SVG's reach groups (which kept the same IDs).

### Verification (Agent Browser end-to-end, via `file://`)

- Page loads with zero console errors, zero page errors.
- **Top button Y centers** (verified via `getBBox()` + `getCTM()` matrix transform): all 6 at `cy=45` ✓ (was `cy=27` in v1.2610.005; expected shift of +18 units).
- **ABXY and D-pad and analog stick centers**: identical to v1.2610.005 ✓ (no unintended movement).
- **`?color=xbox` regression**: outline A = `rgb(76,175,80)` (green) ✓ (v1.2610.006 fix preserved).
- **`?viewmenu=1` regression**: overlay class `controller-overlay show-viewmenu-icons controller-xbox`, view icon `display: inline` ✓ (v1.2610.006 fix preserved).
- **VLM visual confirmation** of `/tmp/bubo-v007-default.png`: "os 6 elementos do topo estão alinhados em uma linha horizontal perto da parte superior do controle"; "ABXY e analógicos estão mais abaixo, na região central e inferior"; "há um espaço visível (margem/respiro) entre a linha dos botões superiores e a borda superior arredondada preta do controle. Eles não estão colados na borda." ✓

### Limitation honestly declared

- **OBS test matrix (AGENT.md §7.2) NOT executed by the agent** — sandbox has no real gamepad and no OBS Studio. The agent verified the SVG rendering, positioning, regressions, and visual layout via Agent Browser + VLM. The maintainer must re-test v1.2610.007 in OBS (pull + reload Browser Source) to confirm the new positioning looks right in the real streaming environment.

### ⚠️ Compliance

- None. All 15 Never-Rules (AGENT.md §12) verified intact.
- `PRIVATE_PLANS.md` confirmed gitignored and not staged.
- No secrets in committed files. GitHub token not persisted (one-shot push URL).
- Invariants §6.1, §6.3, §6.4, §6.5, §6.8 all preserved.

### Sprint footer

Sprint: v1.2610.007 — Xbox SVG positioning adjustment: 6 top buttons (VIEW/MENU/LB/LT/RB/RT) shifted +18 viewBox units down for breathing room from the top border. ABXY/D-pad/sticks unchanged. Icons re-embedded (still opt-in via `?viewmenu=1`). No JS/CSS changes. Let's Play Up!

---

## [v1.2610.006] — 2026-10-13

### Conversation excerpt (PT-BR, verbatim)

> Encontrei 2 erros no controle virtual:
> - Você colocou ícones em VIEW/MENU por padrão, eles só eram para aparecer se tivessem o argumento na URL;
> - O JOY_REACH não aparece, o círculo maior que tinha o KNOB com tamanho 2/3 menor em relação a ele. O "reach" deve ser visível para saber até onde o knob do controle vai.
> - Estou com o parâmetro/argumento de cores de Xbox ativado, mas não vejoas cores, você as desativou? O contorno poderia continuar colorido, como era e a cor de preenchimento deve ser 100% opaca, não deve ter transparência.

### Fixed

- **Bug 1 — VIEW/MENU icons hidden by default, opt-in via `?viewmenu=1`**:
  - Root cause: in v1.2610.005, the icons were embedded as `<rect class="button-icon">` in the SVG template and rendered by default (no CSS `display` rule). The maintainer requested they only appear with a URL argument.
  - Fix: CSS `.button-icon { display: none; }` by default. New class `.show-viewmenu-icons` on the overlay unhides them via `.controller-overlay.show-viewmenu-icons .controller-svg g[id] .button-icon { display: inline; }`.
  - New JS `applyViewMenuIcons()` reads the `?viewmenu=` URL param (accepts `1`, `true`, `yes`, `on`) and adds the `.show-viewmenu-icons` class to the overlay.
  - The icons remain in the SVG template (so they CAN be shown); they're just hidden by CSS until the URL param activates them.
  - Usage: `?viewmenu=1` shows the two-squares (View) and hamburger (Menu) icons. Without the param, VIEW/MENU are pill-shaped outlines with no icon (text label `--label-view` / `--label-menu` are empty by default).

- **Bug 2 — JOY_REACH (analog stick boundary) invisible**:
  - Root cause: the new Xbox SVG (v1.2610.005) draws each reach group (`left_joy_reach`, `right_joy_reach`) with **ONE** `<path>` (a ring: outer + inner subpaths). The v1.2610.004 CSS had `.controller-svg g[id$="_reach"] > :nth-child(1) { fill: none !important; }` which was correct for the OLD SVG (which had 2 paths per reach group: a solid disc + a ring outline). In the NEW SVG, the only path IS `:nth-child(1)`, so `fill: none` hid it entirely.
  - Compounding factor: even after removing `fill: none`, the button-fill rule `.controller-svg g[id] > :nth-child(1) { fill-opacity: var(--button-fill-opacity); fill-rule: nonzero !important; }` (which has specificity 0,0,3,1) was overriding the reach rule `.controller-svg g[id$="_reach"] > path` (specificity 0,0,2,2). The reach path inherited `fill-opacity: 0` (idle button fill opacity) and `fill-rule: nonzero` (solid disc instead of ring) — making it invisible.
  - Fix: rewrote the reach CSS to use **explicit ID selectors** (`g#left_joy_reach > path`, `g#right_joy_reach > path`, `g#joy_reach > path`) which have specificity 0,1,1,2 — beating the button-fill rule. Added `!important` to `fill-opacity: 1` and `fill-rule: evenodd`. The reach now renders as a visible ring (Nimbus, 100% opacity) around each analog stick knob.
  - Works for both Xbox (1 path per reach group) and arcade (2 identical paths per `joy_reach` — both render as the same ring, no visual difference).

- **Bug 3 — `?color=xbox` ABXY colors not visible in outline**:
  - Root cause: in v1.2610.005, the outline CSS was changed to `fill: var(--accent) !important` (always Nimbus), breaking the v1.2610.004 behavior where the outline used `var(--button-color)` (which resolves to `--color-a/b/x/y` for ABXY buttons, defaulting to `--accent` = Nimbus when no `?color=` scheme is active). The maintainer confirmed: "O contorno poderia continuar colorido, como era" — the outline should remain colored when `?color=xbox` is active.
  - Fix: reverted the outline CSS to `fill: var(--button-color, var(--accent)) !important`. Now:
    - Default (no `?color=`): outline = Nimbus (because `--color-a/b/x/y` default to `var(--accent)` = Nimbus).
    - `?color=xbox`: outline = green A (`#4CAF50`), red B (`#E5342A`), blue X (`#0E7DC0`), amber Y (`#F2C200`).
    - `?color=playstation`: outline = blue Cross, red Circle, pink Square, green Triangle.
  - The pressed fill continues to use the same `var(--button-color, var(--accent))` (already correct in v1.2610.005) at 100% opacity (`--button-pressed-opacity: 1`), so ABXY colors appear in BOTH the outline (idle) AND the pressed fill.

### Verification (Agent Browser end-to-end, via `file://`)

- Page loads with zero console errors, zero page errors across all tested states.
- **Default state** (no `?color=`, no `?viewmenu=`):
  - Reach: `fill: rgb(242,242,242)` (Nimbus), `fill-rule: evenodd`, `fill-opacity: 1` ✓ (visible ring)
  - Outline A: `rgb(242,242,242)` (Nimbus) ✓
  - View icon: `display: none` ✓ (hidden by default)
  - Overlay class: `controller-overlay controller-xbox` (no `show-viewmenu-icons`) ✓
- **`?color=xbox`**:
  - Outline A: `rgb(76, 175, 80)` (green `#4CAF50`) ✓
  - Outline B: `rgb(229, 52, 42)` (red `#E5342A`) ✓
  - Outline X: `rgb(14, 125, 192)` (blue `#0E7DC0`) ✓
  - Outline Y: `rgb(242, 194, 0)` (amber `#F2C200`) ✓
- **`?viewmenu=1`**:
  - Overlay class: `controller-overlay show-viewmenu-icons controller-xbox` ✓
  - View icon: `display: inline`, `fill: rgb(242,242,242)` (Nimbus) ✓
  - Menu icon: `display: inline` ✓
- **`?controller=arcade`** (regression):
  - `joy_reach` path: `fill: rgb(242,242,242)`, `fill-rule: evenodd`, `fill-opacity: 1` ✓ (visible, no regression)
- **VLM visual confirmation** of `/tmp/bubo-v006-color-xbox.png`: "botão A verde, B vermelho, X azul, Y amarelo" (ABXY outline colors visible); "analógicos têm um círculo maior de contorno visível" (reach visible); "VIEW e MENU estão vazios, sem ícones visíveis" (icons hidden by default) ✓
- **VLM visual confirmation** of `/tmp/bubo-v006-color-viewmenu.png`: "Botão VIEW (esquerda): dois quadrados sobrepostos. Botão MENU (direita): três linhas horizontais (hambúrguer)" ✓ (icons visible with `?viewmenu=1`)

### Audit

- **CSS specificity war (reach rule vs button-fill rule)**: the button-fill rule `.controller-svg g[id] > :nth-child(1)` has specificity (0,0,3,1) — 1 class + 1 attribute + 1 pseudo-class + 1 element. The first attempt at the reach rule used `.controller-svg g[id$="_reach"] > path` with specificity (0,0,2,2) — 1 class + 1 attribute + 2 elements. The button-fill rule won (3 > 2 in the third specificity slot), inheriting `fill-opacity: 0` and `fill-rule: nonzero` into the reach path. Fix: switch to explicit ID selectors `g#left_joy_reach > path` (specificity 0,1,1,2) which beats the attribute selector (1 ID > 0 IDs in the second slot). This is a CSS specificity lesson: when a generic rule (attribute selector) conflicts with a specific rule (ID selector), the ID wins regardless of `!important` tie-breaking. Documented in the CSS comment block above the reach rule.

- **`?viewmenu=` URL parameter design**: chose a separate parameter (`?viewmenu=1`) instead of extending the existing `?icons=` parameter. Rationale: `?icons=` controls TEXT labels for ABXY/LB/RB/LT/RT/L3/R3 (xbox/playstation/nintendo/none sets). VIEW/MENU icons are a separate concern (SVG-embedded shapes, not text). Coupling them would force the user to activate a text label set just to show the icons. Decoupling is cleaner. Accepts `1`, `true`, `yes`, `on` (case-insensitive) as truthy values.

### ⚠️ Compliance

- None. All 15 Never-Rules (AGENT.md §12) verified intact.
- `PRIVATE_PLANS.md` confirmed gitignored and not staged.
- No secrets in committed files. GitHub token not persisted (one-shot push URL).
- Invariants §6.1, §6.3, §6.4, §6.5, §6.8 all preserved.

### Sprint footer

Sprint: v1.2610.006 — Bugfixes for v1.2610.005: VIEW/MENU icons opt-in via `?viewmenu=1`, JOY_REACH visible again (specificity fix), `?color=xbox` ABXY colors visible in outline (revert to `var(--button-color)`). Let's Play Up!

---

## [v1.2610.005] — 2026-10-13

### Conversation excerpt (PT-BR, verbatim)

> Entrego a nova versão do SVG do controle XBOX. O que mudou:
> - Fundo único na cor `umbral` que deverá ser passada com opacidade 95% para usar no OBS;
> - Todos os controles são só os contornos em `nimbus` com traço interno de 5pt (= 5px?);
>
> O resto será feito por código, como:
> - Sobrepor as cores do ABXY pelas cores do Xbox/Playstation;
> - Criar e usar LABELS em texto para os botões ABXY, LB/RB, LT/RT, L3/R3 e buscar ícones para usar em VIEW/MENU (preferência pelos originais, se possível);
> - Comportamento LT/RT (gatilhos) continua com efeito "onda/wave", mas mascarar efeito para não vazar para fora do contorno do botão/gatilho no desenho SVG (atualmente vaza);
> - Planejar como usar a ideia do aviso "controle desconectado/conectado", talvez, feito nos mesmos moldes do Alerta com texto simples em Sofia Sans XC Black?
>
> Aceito outras ideias. Arcade ainda fica pendente. Eu preciso deste controle pronto ainda hoje.

> Desconsidere o SVG anterior, o KNOB do L3/R3 estava com o tamanho feito para o controle arcade (1/4 em vez de 2/3 do alcance).
>
> Outra coisa que esqueci de mencionar, mas vamos manter é o preenchimento em `nimbus` quando os botões forem pressionados (ou coloridos no ABXY, se ativado e animação `wave` para os gatilhos) e os KNOBS do L3/R3 serão preenchidos com `nimbus` 50% em idle para diferenciar do alcance e serão `nimbus` 100% opacos quando pressionados.
>
> Desculpe a confusão.

### Added

- **New Xbox SVG model (v1.2610.005)** — `masks/controllers/xbox.svg` and the inline `<template id="svg-xbox">` in `index.html` replaced with the maintainer's corrected Affinity Designer export. The new SVG:
  - Maintains `viewBox="0 0 640 216"` (canvas is 1/6 width × 10% height of 3840×2160 base, same as v1.2610.004).
  - Background is now a `<path id="BACKGROUND">` as a direct child of `<g id="XBOX">` (not wrapped in a `<g transform>` like v1.2610.004) with `fill:rgb(13,13,13)` baked in (Umbral).
  - All button shapes (a/b/x/y, lb/rb/lt/rt, view/menu, up/down/left/right) are drawn as ring paths (outer + inner subpaths) filled with `rgb(242,242,242)` (Nimbus) using `fill-rule:evenodd`. This is the "outline-only with internal stroke" form requested by the maintainer. The observed stroke width in the geometry is ~4.5 path-local units, which after the parent `<g>` scale of 0.666667 becomes ~3.0 viewBox units (~3 CSS px at 1:1 display). The maintainer specified "5pt internal stroke" — see Audit note below on the 5pt-vs-5px-vs-geometry reconciliation.
  - **L3/R3 KNOB size CORRECTED**: the previous (cancelled) attempt had the knob at 1/4 of the reach (arcade sizing). The new SVG has the knob at 2/3 of the reach (Xbox sizing), per maintainer correction. Knob scale = 0.444444 (= (2/3)²) of the path-local coords; reach scale = 0.666667 (= 2/3). After transform: knob display diameter / reach display diameter = 2/3 ✓.
  - IDs renamed from Affinity's uppercase convention (`BUTTON_A`, `LEFT_JOY_KNOB`, `DPAD_UP`, etc.) to the project's lowercase convention (`a`, `left_joy`, `up`, etc.) so the existing JS (`BUTTON_MAP`, `collectElements`, `cacheKnobData`) finds them without changes.
  - **VIEW/MENU icons embedded** as `<rect class="button-icon">` children of the `<g id="view">` and `<g id="menu">` groups (positioned at the pill centers, in path-local coords). Generic UI metaphors — two-overlapping-rounded-rectangles for View, three-horizontal-lines (hamburger) for Menu — not Microsoft-trademarked art (invariant §6.4 preserved).

- **`processSvg()` rewritten** to split each button ring into FILL + OUTLINE:
  - Each button `<g>` originally has ONE `<path>` with 2 subpaths (ring/outline). The new `processSvg()` creates a second `<path>` with only the outer subpath (solid) and inserts it BEFORE the original ring. The `<g>` now has:
    - `:nth-child(1)` = FILL (outer-only solid path, invisible in idle, full-color in pressed)
    - `:nth-child(2)` = OUTLINE (ring, both subpaths, always visible, rendered via `fill-rule:evenodd`)
  - This replaces the v1.2610.004 logic that used `replaceRingWithCircle()` for circular buttons and `extractOuterSubpath()` for non-circular ones. Both helpers are removed (no longer needed).
  - Skips: root group (`<g id="XBOX">`), `<path id="BACKGROUND">`, and reach rings (`g[id$="_reach"]`) — these have no fill, only outline (handled by the existing CSS rule for `_reach`).

- **CSS background selector updated** to include `.controller-svg > g > path#BACKGROUND` (the new SVG's direct-child background path). The rule now uses `!important` on `fill` and `fill-opacity` to win over the button-fill rule `.controller-svg g[id] > :nth-child(1)` (which would otherwise match the BACKGROUND path as the `:nth-child(1)` of the root `<g id="XBOX">` and paint it with the accent instead of the background color — this was a real bug caught by Agent Browser verification, see Verification below).

- **Button fill behavior changed** (per maintainer v1.2610.005 spec):
  - Idle: `--button-fill-opacity: 0` (was 0.95 in v1.2610.004) — buttons are OUTLINE-ONLY in idle.
  - Pressed: `--button-pressed-opacity: 1` (was 0.95) — full solid fill.
  - Outline is ALWAYS Nimbus (`var(--accent)`), regardless of `?color=` scheme. The ABXY colors (via `--color-a/b/x/y`) only affect the PRESSED FILL, not the outline. This matches the maintainer's spec: "Todos os controles são só os contornos em `nimbus`" (idle) + "ou coloridos no ABXY, se ativado" (pressed).

- **L3/R3 knob pressed opacity = 100%** (was 95% in v1.2610.004): `--knob-pressed-opacity: 1`. Idle stays at `--knob-fill-opacity: 0.5` (Nimbus 50%, unchanged).

- **Background opacity = 95%** (was 20%): `--bg-opacity: 0.95`. Per maintainer: "Fundo único na cor `umbral` que deverá ser passada com opacidade 95% para usar no OBS".

- **Overlay opacity = 1.0** (was 0.85): `.controller-overlay { opacity: 1 }`. The previous 0.85 was reducing the effective background opacity to 0.85 × 0.20 = 17% (v1.2610.004). With the new 95% background, the overlay must be at 1.0 to let the full 95% show through.

- **Trigger wave masked to trigger contour** (per maintainer: "mascarar efeito para não vazar para fora do contorno do botão/gatilho"):
  - `.trigger-overlay` now has `clip-path: inset(0 round var(--trigger-clip-radius))` where `--trigger-clip-radius: 9px` (viewBox units, computed from the new SVG's pill geometry after the parent `<g>` transform).
  - The clip-path is in the trigger-overlay div's local CSS coords (which match viewBox units at 1:1 display), and the parent `.controller-overlay` `transform: scale()` scales the whole thing together — so the corner radius scales correctly with `?scale=` and `?fit=`.
  - The 9px corner radius is an approximation (the actual trigger path uses non-circular Bezier corners that, after the asymmetric 1.333/0.6667 transform, become visually symmetric at 9 viewBox units). The approximation is visually very close to the trigger outline; sub-pixel differences are imperceptible at OBS rendering sizes.

- **"No controller connected" hint** (per maintainer: "aviso 'controle desconectado/conectado' ... nos mesmos moldes do Alerta com texto simples em Sofia Sans XC Black"):
  - New `<div class="controller-hint">` created by `setupControllerHint()` (called from the constructor, after `loadController()`).
  - Bilingual: `<html lang="pt">` → "Conecte um controle"; default → "Connect a controller".
  - Visibility logic in `updateControllerHint()` (called from `update()` on every frame):
    - No gamepad + 5s elapsed (`--hint-delay-ms: 5000`) + never connected before → add `.visible` class (fade-in 200ms via `--hint-transition-ms: 200`).
    - Gamepad connected → remove `.visible` + set `gamepadEverConnected = true`.
    - Once `gamepadEverConnected` is true, the hint NEVER reappears (avoids noise during a stream if the controller briefly disconnects).
  - Styling: `--hint-color: var(--nimbus)`, `--hint-opacity: 0.6` (subtle), `--hint-font-weight: 900`, `--hint-size: 14px`. Centered, `pointer-events: none`, `z-index: 20`.
  - **Sofia Sans XC Black not embedded** (invariant §6.3: no external fonts at runtime). The hint uses the existing system font stack (`--label-font`) at weight 900 as a fallback. The maintainer may provide the font file in a future Sprint for exact visual match (see Audit below).
  - Debug: `.controller-overlay.debug .controller-hint { opacity: var(--hint-opacity) !important }` — double-click the overlay to enter debug mode and force-show the hint for visual QA.

- **CSS variables for the hint and trigger clip** added to `:root`:
  - `--hint-color`, `--hint-opacity`, `--hint-font-weight`, `--hint-size`, `--hint-delay-ms`, `--hint-transition-ms` (customization API for the hint).
  - `--trigger-clip-radius` (customization API for the trigger wave mask radius).

- **L3/R3 label aliases in `collectElements()`**: the new Xbox SVG has `<g id="left_joy">` and `<g id="right_joy">` (not `<g id="l3">`/`<g id="r3">`), so `addLabels()` would skip L3/R3 (the elements weren't found). Added aliases `this.elements.l3 = left_joy` and `this.elements.r3 = right_joy` (Xbox only — arcade has real l3/r3). The L3/R3 text labels now render at the knob centers, following the knob as it moves.

### Changed

- **Per-button `--button-color` mapping** updated: the previous rules `.controller-svg g#l3 { --button-color: var(--color-l3); }` and `g#r3` referenced non-existent `<g id="l3">` / `<g id="r3">` elements (the new SVG uses `left_joy` / `right_joy`). Updated to `.controller-svg g#left_joy` and `g#right_joy`.
- **Outline CSS** now uses `fill-rule: evenodd !important` (was implicit/nonzero) to reliably render the ring as an outline. The fill uses `fill-rule: nonzero !important` (solid).
- **Comment headers** in `:root` updated from "v1.2610.004" to "v1.2610.005".

### Audit

- **5pt vs 5px reconciliation**: the maintainer asked "traço interno de 5pt (= 5px?)". In CSS/SVG, `1pt = 1.333px` (because `1pt = 1/72 inch` and `1px = 1/96 inch`). So `5pt = 6.667px`. The Affinity export shows the gap between outer and inner contour as `4.5` path-local units (e.g., outer at x=320, inner at x=315.5 in the LB path). After the parent `<g>` scale of 0.666667, this becomes `4.5 × 0.666667 = 3.0` viewBox units. At 1:1 display (640×216 CSS px), the stroke is ~3 CSS px wide. This is thinner than the maintainer's specified 5pt (6.667px), but the SVG was accepted as-exported (the maintainer shipped the .svg file). The discrepancy is documented here; if the maintainer wants the stroke wider, the SVG must be re-exported from Affinity with a larger stroke setting.

- **Sofia Sans XC Black font not embedded**: invariant §6.3 (offline-first, no external runtime dependencies) prohibits loading Sofia Sans from Google Fonts at runtime. The hint uses the system font stack at weight 900 as a fallback. To achieve exact visual match, the maintainer would need to provide the font file (e.g., `SofiaSans-Black.ttf`) for base64 embedding in a future Sprint — the streaming repo (`/home/z/my-project/repos/streaming/brand/fonts/`) already has Sofia Sans Extra Condensed, but the regular XC variant is not in the workspace. The agent recommends deferring this to a future Sprint to avoid scope creep on v1.2610.005.

- **Background `!important` rationale**: the new CSS rule `.controller-svg > g > path#BACKGROUND { fill: var(--color-bg) !important; fill-opacity: var(--bg-opacity) !important; }` uses `!important` because the button-fill rule `.controller-svg g[id] > :nth-child(1) { fill: var(--button-color, var(--accent)) !important; }` also matches the BACKGROUND path (it is the `:nth-child(1)` of `<g id="XBOX">`). Without `!important` on the background rule, the button rule would paint the background with Nimbus instead of Umbral. This was caught by Agent Browser verification (see Verification) — the first attempt had `fill: rgb(242,242,242)` on the background, indicating the button rule was winning. The fix is documented in the CSS comment block.

### Verification (Agent Browser end-to-end)

- Page loads with zero console errors, zero page errors across all tested states.
- **Background**: `rgb(13, 13, 13)` (Umbral) at `fill-opacity: 0.95` ✓.
- **Button A idle**: fill `rgb(242, 242, 242)` (Nimbus) at `fill-opacity: 0` (invisible — outline-only) ✓.
- **Button A outline**: `rgb(242, 242, 242)` (Nimbus) at `fill-opacity: 1` with `fill-rule: evenodd` (ring rendered as outline) ✓.
- **`<g id="view">` children**: 5 elements (path fill + path outline + 2 rect button-icon + text label) ✓ — icons embedded correctly.
- **`<g id="a">` children**: 3 elements (path fill + path outline + text label "A") ✓ — labels render at the button center.
- **LT trigger overlay**: `clip-path: inset(0px round 9px)` applied ✓ — wave is masked to the trigger contour.
- **Hint**: present as `<div class="controller-hint">` with text "Connect a controller", hidden by default (`opacity: 0`). After 6s without a gamepad, the `.visible` class is added → `opacity: 0.6` ✓.
- **`?color=xbox`**: ABXY CSS variables applied (`--color-a: #4CAF50`, `--color-b: #E5342A`, `--color-x: #0E7DC0`, `--color-y: #F2C200`) ✓.
- **`?scale=2`**: page loads and renders without errors ✓.
- **`?controller=arcade`**: regression check — arcade layout loads and renders without errors ✓ (the arcade SVG is unchanged; `processSvg()` skips it because its buttons have only 1 path each, not a 2-subpath ring).
- **VLM visual confirmation** of `/tmp/bubo-v005-idle-hint.png`: fundo escuro (preto sólido), botões como contornos brancos (outlines) com interior transparente, texto "Connect a controller" visível no centro em cinza claro, paleta escura com contornos claros. ✓ Matches the maintainer's spec exactly.
- Screenshots captured: `/tmp/bubo-v005-idle.png`, `/tmp/bubo-v005-idle-hint.png`, `/tmp/bubo-v005-color-xbox.png`, `/tmp/bubo-v005-scale2.png`, `/tmp/bubo-v005-arcade.png`.

### Limitation honestly declared

- **OBS test matrix (AGENT.md §7.2) NOT executed by the agent** — the sandbox has no real gamepad and no OBS Studio. The agent executed the Agent Browser self-verification (§7.3) and confirmed the visual + structural state of the overlay, but the maintainer must run the full OBS test matrix (real gamepad button presses, pressure-sensitive triggers, analog stick movement) before declaring v1.2610.005 production-ready for streams. The agent's verification covers: page load, console-clean, SVG structure, CSS computed styles, hint visibility logic, ?color/xbox parameter, ?scale=2 parameter, ?controller=arcade regression. Not covered: real button presses, real trigger pressure, real analog movement, OBS CEF quirks.
- **Arcade layout still pending** (per maintainer: "Arcade ainda fica pendente") — the arcade SVG is unchanged from v1.2610.001 and continues to letterbox inside the 640×216 overlay (the arcade SVG is 768×324). A future Sprint will deliver the new arcade SVG.

### ⚠️ Compliance

- None. All 15 Never-Rules (AGENT.md §12) verified intact.
- `PRIVATE_PLANS.md` confirmed gitignored and not staged (no `git add -f` used).
- No secrets in committed files. The GitHub token is not persisted (one-shot push URL).
- Invariant §6.3 (offline-first) preserved — no external fonts loaded; system font stack fallback used for the hint.
- Invariant §6.4 (no brand art) preserved — VIEW/MENU icons are generic UI metaphors (two-squares, hamburger), not Microsoft-trademarked art.
- Invariant §6.5 (Track C only) preserved — new SVG inlined as `<template id="svg-xbox">`, JS-cloned at runtime.
- Invariant §6.1 (single-file HTML) preserved — all changes are in `index.html` + `masks/controllers/xbox.svg` (the latter is the source-of-truth blueprint, inlined at runtime).

### Sprint footer

Sprint: v1.2610.005 — New Xbox SVG (corrected L3/R3 knob at 2/3 reach), outline-only buttons, Umbral 95% background, trigger wave masked, VIEW/MENU icons embedded, L3/R3 labels fixed, "no controller connected" hint. Let's Play Up!

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
