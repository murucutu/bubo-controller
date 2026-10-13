# AUDIT.md

> **Nota histórica (v1.2610.002):** Este documento é o registro interno de auditoria da arquitetura do projeto. As entradas abaixo descrevem a evolução **Track A/B → Track C**. A arquitetura atual (Track C) é **SVG inline** (sem `masks/bg_dark.png`, sem crop CSS); os trechos que mencionam `bg_dark.png`, `background-image` e `--crop-x`/`--crop-y` são registros do estado **anterior** e foram preservados como histórico. Para o estado atual da arquitetura, consulte [VARIANTS.md](../VARIANTS.md), [DESIGN_GUIDE.md](../DESIGN_GUIDE.md) e o [CHANGELOG.md](../CHANGELOG.md) (especialmente a entrada `v1.2610.001` que removeu o `bg_dark.png`).

## Resumo da execução

- O novo overlay foi consolidado em `index.html`.
- `index2.html` foi usado como base e removido após a migração.
- `style.css` e `controller.js` foram removidos; CSS e JS agora estão inline no HTML.
- Scripts Python e geradores foram removidos por não serem necessários ao runtime OBS.
- Assets de máscara não usados foram removidos; o overlay final usa apenas `masks/bg_dark.png`.
- O erro visual dos analógicos LS/RS foi corrigido com boundary de `172x172` e centralização por CSS.
- A linha irregular dos gatilhos LT/RT foi restaurada como canvas leve, reagindo à pressão sem reintroduzir WebHID.

Arquivos finais:

- `index.html`
- `masks/bg_dark.png`
- `AUDIT_PLAN.md`

---

## 1. Otimizações e Remoção de Código-Lixo

### Remoção de duplicação entre `index.html`, `index2.html`, `style.css` e `controller.js`

- [Arquivo/Linha]: Antes da limpeza, havia leiaute duplicado em `index.html`/`index2.html`, CSS em `style.css` e JS em `controller.js`.
- [Solução]: Consolidar tudo em `index.html`.
  - Antes: `index.html` dependia de `style.css` e `controller.js`; `index2.html` mantinha outro leiaute inline.
  - Depois: `index.html` contém HTML, CSS e JS autocontidos; `index2.html`, `style.css` e `controller.js` foram removidos.

### Remoção de scripts Python e geradores desnecessários

- [Arquivo/Linha]: Antes da limpeza, existiam `analyze_masks.py`, `generate_overlay_uv.py`, `generate_overlay.py`, `generate_css.py` e `positions.json`.
- [Solução]: Remover scripts Python e arquivos de geração porque o overlay OBS não precisa gerar assets em runtime.
  - Antes: projeto dependia de pipeline Python para gerar HTML/CSS/JS/posições.
  - Depois: projeto final tem apenas `index.html` e `masks/bg_dark.png`.

### Centralização dos analógicos e remoção de CSS duplicado

- [Arquivo/Linha]: `index2.html:308-342` tinha regras duplicadas para `.ls-boundary`, `.rs-boundary`, `.analog-knob.ls` e `.analog-knob.rs`.
- [Solução]: Unificar `.analog-knob.ls/.rs` em `.analog-knob` e centralizar com `left/top: 50%`.
  - Antes:
    ```css
    .ls-boundary {
        left: 0;
        top: 120px;
        width: 192px;
        height: 192px;
    }

    .analog-knob.ls {
        width: 114.6667px;
        height: 114.6667px;
        left: 38.6667px;
        top: 38.6667px;
    }
    ```
  - Depois:
    ```css
    :root {
        --analog-max-offset: 28.6667px;
    }

    .analog-knob {
        position: absolute;
        left: 50%;
        top: 50%;
        width: 66.6667%;
        height: 66.6667%;
        transform: translate(calc(-50% + var(--stick-x, 0px)), calc(-50% + var(--stick-y, 0px)));
    }

    .ls-boundary {
        left: 10px;
        top: 120px;
        width: 172px;
        height: 172px;
    }

    .rs-boundary {
        left: 586px;
        top: 120px;
        width: 172px;
        height: 172px;
    }
    ```

### Redução de complexidade ciclomática no JavaScript

- [Arquivo/Linha]: `index2.html:477-884` continha WebHID, parsing de relatórios, calibração, canvas de gatilho e toque.
- [Solução]: Reduzir o JS inline para Gamepad API, botões, pressão dos gatilhos, analógicos e um canvas leve apenas para o gráfico dos gatilhos.
  - Antes: múltiplos métodos para WebHID, parsing de triggers, canvas e toque.
  - Depois: `index.html:409-653` mantém apenas o fluxo necessário para OBS Browser Source.

---

## 2. Robustez, Tratamento de Erros e Prevenção

### Correção da sobrescrita de `transform` pelo JavaScript

- [Arquivo/Linha]: Antes, `index2.html:767-773` sobrescrevia `knob.style.transform`, quebrando a centralização estática do knob.
- [Solução]: Separar centralização CSS do deslocamento dinâmico por variáveis CSS.
  - Antes:
    ```js
    knob.style.transform = `translate(${normalizedX * ANALOG_MAX_OFFSET}px, ${normalizedY * ANALOG_MAX_OFFSET}px)`;
    ```
  - Depois:
    ```js
    knob.style.setProperty('--stick-x', `${normalizedX * maxOffset}px`);
    knob.style.setProperty('--stick-y', `${normalizedY * maxOffset}px`);
    ```

### Reset explícito do estado neutro

- [Arquivo/Linha]: `index.html:504-513` e `index.html:548-558`.
- [Solução]: Garantir que LS/RS voltem para `0px` quando não houver gamepad.
  - Antes: o reset dependia de sobrescrever `transform` inline.
  - Depois: `resetAnalogSticks()` define `--stick-x: 0px` e `--stick-y: 0px` para ambos os knobs.

### Prevenção contra gatilhos travados sem controle conectado

- [Arquivo/Linha]: `index.html:548-558`.
- [Solução]: Resetar pressão dos gatilhos ao perder o gamepad ativo.
  - Antes: a remoção do gamepad limpava botões, mas podia manter pressão visual dos gatilhos.
  - Depois:
    ```js
      this.setTriggerPressure('lt', 0);
      this.setTriggerPressure('rt', 0);
      ```

### Correção da pressão total dos gatilhos LT/RT

- [Arquivo/Linha]: `index.html:469-480`.
- [Solução]: Triggers analógicos agora retornam o valor real quando `button.value > 0`; triggers digitais/totalmente pressionados retornam `1`, não `0.40`.
  - Antes:
    ```js
    if (value > 0 && value < 1) {
        return value;
    }

    return button.pressed ? 0.40 : 0;
    ```
  - Depois:
    ```js
    const value = clamp(button.value);
    if (value > 0) {
        return value;
    }

    return button.pressed ? 1 : 0;
    ```

### Remoção de ruído de WebHID e manutenção do canvas de gatilho

- [Arquivo/Linha]: Antes, `index2.html` continha parsing de WebHID e desenho contínuo de canvas.
- [Solução]: Remover WebHID, mantendo apenas um canvas leve para a linha irregular dos gatilhos. O overlay usa `navigator.getGamepads()` para estado e pressão.
  - Antes: parsing heurístico de relatórios e desenho de grafos em canvas a cada frame.
  - Depois: `index.html:461-467` busca gamepad ativo, `index.html:597-637` desenha o gráfico e `index.html:639-643` atualiza o estado.

---

## 3. SOTA (Estado da Arte), Tipagem e Padronização

### HTML/CSS/JS unificado para OBS Browser Source

- [Arquivo/Linha]: `index.html:1-659`.
- [Solução]: Usar um único arquivo HTML autocontido.
  - Antes: `index.html`, `index2.html`, `style.css`, `controller.js` e geradores Python.
  - Depois: `index.html` único, com CSS inline em `<style>` e JS inline em `<script>`.

### Padronização de nomes

- [Arquivo/Linha]: `index.html:409-653`.
- [Solução]: Nomes de métodos e propriedades seguem o fluxo simples do overlay: `getGamepad`, `updateGamepadState`, `updateAnalogState`, `resetAnalogSticks`, `setTriggerPressure`.
  - Antes: nomes e responsabilidades misturavam overlay visual, WebHID, canvas e geração de assets.
  - Depois: classe `XboxControllerOverlay` com responsabilidade única de ler Gamepad API e atualizar DOM/CSS.

### PEP 8 e type hints em Python

- [Arquivo/Linha]: scripts Python removidos.
- [Solução]: Não há mais código Python no runtime, portanto PEP 8 e type hints não se aplicam ao projeto final.
  - Antes: `analyze_masks.py`, `generate_overlay_uv.py`, `generate_overlay.py` e `generate_css.py` exigiam auditoria Python.
  - Depois: projeto final não usa Python.

### Design Patterns

- [Arquivo/Linha]: `index.html:409-653`.
- [Solução]: Manter uma classe simples de fachade para o overlay. Não há acoplamento ruim suficiente para justificar padrão mais complexo.
  - Antes: código espalhado entre geradores, CSS externo, JS externo e lógica WebHID.
  - Depois: responsabilidade única concentrada em `XboxControllerOverlay`.

---

## 4. Lazy Processing e Alocação de Recursos

### Restauração leve do canvas dos gatilhos

- [Arquivo/Linha]: `index.html:219-223`, `index.html:350-354`, `index.html:423-443` e `index.html:597-637`.
- [Solução]: Reintroduzir apenas o canvas da linha irregular dos gatilhos, sem WebHID, sem parsing complexo e sem toque.
  - Antes da restauração: o gráfico foi removido junto com o código excedente.
  - Depois:
    ```html
    <canvas class="trigger-graph" width="185" height="62" aria-hidden="true"></canvas>
    ```
    ```js
    this.drawTriggerGraph('lt');
    this.drawTriggerGraph('rt');
    ```

### Redução de trabalho por frame

- [Arquivo/Linha]: `index.html:639-643`.
- [Solução]: Cada frame lê gamepads, atualiza estados e desenha os dois canvases pequenos dos gatilhos.
  - Antes: frame incluía update de botões, analógicos, WebHID, canvas e toque.
  - Depois: frame inclui update de botões, pressão dos gatilhos, analógicos e desenho leve dos gatilhos.

### Recurso mantido: polling por `requestAnimationFrame`

- [Arquivo/Linha]: `index.html:645-649`.
- [Solução]: Manter `requestAnimationFrame` para detectar entrada em tempo real no OBS.
  - Antes: `requestAnimationFrame` executava trabalho pesado adicional.
  - Depois: `requestAnimationFrame` executa polling leve da Gamepad API e desenho de dois canvases pequenos.

---

## 5. SSOT (Single Source of Truth) e Hardcoding

### Assets

- [Arquivo/Linha]: `index.html:54-60`.
- [Solução]: O único asset usado pelo runtime é `masks/bg_dark.png`.
  - Antes: múltiplos assets de máscara e geradores.
  - Depois:
    ```css
    background-image: url('masks/bg_dark.png');
    background-size: var(--base-width) var(--base-height);
    background-position: calc(var(--crop-x) * -1) calc(var(--crop-y) * -1);
    ```

### Constantes centralizadas no CSS

- [Arquivo/Linha]: `index.html:8-18`.
- [Solução]: Centralizar dimensões, crop, stroke, cor e offset dos analógicos em `:root`.
  - Antes: valores espalhados entre CSS inline, `style.css`, `positions.json` e geradores.
  - Depois:
    ```css
    :root {
        --base-width: 3840px;
        --base-height: 2160px;
        --overlay-width: 768px;
        --overlay-height: 324px;
        --crop-x: 1536px;
        --crop-y: 1728px;
        --stroke: 5px;
        --accent: #F2F2F2;
        --analog-max-offset: 28.6667px;
    }
    ```

### Números mágicos e strings hardcoded restantes

- [Arquivo/Linha]: `index.html:117-313`.
- [Solução]: As posições visuais continuam inline porque o projeto final é um único HTML. Para evitar regressão futura, manter essas coordenadas agrupadas no bloco CSS do overlay.
  - Exemplos relevantes:
    - `185px`, `62px`, `12px` para LT/RT.
    - `92px`, `58px`, `54px`, `80px` para botões centrais e D-pad.
    - `261.02px`, `208px`, `288px`, `189.02px` para D-pad.
    - `451.25px`, `502.5px`, `400px`, `57.5px` para ABXY.
    - `172px`, `10px`, `586px`, `120px` para analógicos.
- [Arquivo/Linha]: `index.html:469-480`.
- [Solução]: Constantes de comportamento permanecem próximas do uso no JS:
  - `ANALOG_DEAD_ZONE = 0.05`
  - retorno completo para trigger digital/totalmente pressionado `1`
  - limiar de botão analógico `0.1`
  - limiar de calibração neutra `0.20`
  - fallback de offset `28.6667`

### Remoção de SSOT legado

- [Arquivo/Linha]: `positions.json` removido.
- [Solução]: Como não há mais gerador, `positions.json` deixou de ser necessário.
  - Antes: posições eram geradas/consultadas por script Python.
  - Depois: `index.html` é a fonte única do runtime visual.

---

## 6. Roadmap de Evolução

### Curto prazo

- Abrir `index.html` no OBS Browser Source.
- Confirmar LS/RS centralizados sem controle conectado.
- Conectar gamepad e validar movimento dos analógicos.
- Ajustar `--analog-max-offset` apenas se o knob precisar de mais ou menos curso visual.

### Médio prazo

- Se novas posições forem necessárias, criar um único bloco de constantes CSS no topo do arquivo.
- Evitar reintroduzir geradores Python no runtime.
- Manter o projeto como um único arquivo HTML enquanto o OBS for o alvo principal.

### Longo prazo

- Adicionar testes visuais automatizados se o projeto voltar a crescer.
- Considerar separação JS/CSS apenas se houver necessidade real de reutilização fora do OBS.
- Documentar dimensões do crop e offset dos analógicos no próprio arquivo para evitar regressão visual.

---

## Validação executada

- `Get-ChildItem -Recurse -File` confirmou arquivos finais:
  - `AUDIT_PLAN.md`
  - `index.html`
  - `masks/bg_dark.png`
- `Select-String` em `index.html` não encontrou referências a:
  - `index2.html`
  - `style.css`
  - `controller.js`
  - `WebHID`
  - `generate_`
  - `positions`
  - `analyze_masks`
  - `print(`
  - `0.40`
- `Select-String` confirmou que `canvas` aparece apenas como `.trigger-graph` intencional dos gatilhos.
- `Test-Path .\masks\bg_dark.png` confirmou que o asset exigido pelo background existe.
- Checagem de sintaxe JS via `node --check` não foi executada porque Node.js não está instalado no ambiente.

---

## v1.2610.004 — Novo modelo Xbox 640×216, paleta Nimbus/Umbral, L3/R3 especial

### Contexto

O mantenedor entregou um novo `masks/controllers/xbox.svg` exportado do Affinity Designer, com `viewBox="0 0 640 216"` (1/6 da largura × 10% da altura de uma base 3840×2160). Junto ao novo SVG, pediu:
- Fundo a 20% de opacidade.
- Contorno `Nimbus` (#F2F2F2) e fundo `Umbral` (#0D0D0D) para todos os botões.
- Botões com 95% de opacidade por padrão.
- L3 e R3 com `Nimbus 50%` como base.
- Pressionados: botões mudam para `Nimbus 95%` (ou a cor do esquema Xbox se `?color=xbox` ativo).
- Preservar a onda dos gatilhos LT/RT.
- Arcade: entregará novo SVG em Sprint futura.

### Decisões adotadas pelo agente (não-invariantes, per AGENT.md §11.5)

1. **`view1` → `menu` rename**: o Affinity Designer auto-sufixou o segundo layer chamado `view` para `view1` com `affinity:id="view"`. O agente renomeou para `menu` e removeu o atributo `affinity:id` para que o JS `BUTTON_MAP.menu = 9` continue mapeando o Gamepad button 9. Sem essa renomeação, o botão Menu não reagiria. Documentado como default técnico não-invariante.

2. **L3/R3 interpretação (a)**: o mantenedor não explicitou se L3/R3 seriam (a) knob do analógico com Nimbus 50% base (stick-click indicator) ou (b) elementos SVG separados. O agente adotou (a) porque:
   - É consistente com a convenção Xbox documentada em `masks/controllers/README.md`: "L3/R3 in the Xbox layout are the stick-click (pressing LS/RS knobs)".
   - O JS existente (linhas 1008-1011 do `index.html`) já faz `left_joy.pressed` / `right_joy.pressed` quando os botões 10/11 são pressionados, confirmando que L3/R3 = stick-click.
   - O novo SVG não adiciona elementos `g#l3` / `g#r3` separados (apenas `left_joy` / `right_joy`), então (b) exigiria redesign do SVG.
   A implementação: `--color-l3` e `--color-r3` apontam para `--nimbus` (não `--accent`), e regras CSS dedicadas `.controller-svg g#left_joy.pressed > :nth-child(1)` aplicam `fill: var(--color-l3)` a `--knob-pressed-opacity` (0.95) quando pressionado. L3/R3 mantêm Nimbus mesmo sob `?color=xbox` (não seguem o esquema ABXY).

### Mudanças arquiteturais

- **Canvas**: 768×324 → 640×216 (aspect ~2.37:1 → ~2.96:1).
  - `:root`: `--overlay-width: 640px`, `--overlay-height: 216px`.
  - JS: `OVERLAY_BASE_WIDTH = 640`, `OVERLAY_BASE_HEIGHT = 216`.
  - `<template id="svg-xbox">`: substituído pelo novo SVG inline.
  - Root group: `Prancheta1` → `XBOX`.
  - Background: `<rect>` direto → `<path>` (rounded rect) dentro de `<g transform>`, com `fill-opacity:0.2` embutido no SVG (removido pelo `processSvg()`, reaplicado via CSS).

- **Paleta semântica**: introduzidas `--nimbus` (#F2F2F2) e `--umbral` (#0D0D0D) como fonte da verdade; `--accent` e `--color-bg` agora são aliases (`var(--nimbus)` e `var(--umbral)`). Preserva backwards-compat com snippets de customização existentes.

- **Opacidades**:
  - `--bg-opacity: 0.5 → 0.2` (fundo 20%).
  - `--button-fill-opacity: 0.5 → 0.95` (botão 95%).
  - `--button-pressed-opacity: 1 → 0.95` (pressionado mantém 95%, não salta para 100%).
  - `--knob-fill-opacity: 0.4 → 0.5` (L3/R3 base 50% Nimbus).
  - `--knob-pressed-opacity: 1 → 0.95` (L3/R3 pressionado 95% Nimbus).

- **Seletor CSS de background**: `.controller-svg > g > rect` estendido para `.controller-svg > g > rect, .controller-svg > g > g:first-child > path` para casar com ambos os templates (arcade ainda usa `<rect>`; xbox agora usa `<path>` dentro de `<g>`).

- **Labels**: `--label-size: 22px → 18px`, `--label-size-small: 16px → 13px` (proporcionais ao canvas menor).

### Preservado (sem mudança)

- **Onda LT/RT**: `setupTriggerOverlays()` + `drawTriggerGraph()` (canvas) — sem mudança. Verificado: 2 trigger overlays presentes via Agent Browser.
- **Arcade layout**: `arcade.svg` e `<template id="svg-arcade">` — sem mudança (ainda 768×324). Letterbox dentro do overlay 640×216 até o mantenedor entregar o novo SVG arcade.

### Verificação Agent Browser

- Zero erros de console, zero erros de página.
- `--overlay-width: 640px`, `--overlay-height: 216px` (computed).
- Background fill: `rgb(13,13,13)` (Umbral) a 20%.
- Botão A base: fill `rgb(13,13,13)` (Umbral) a 95%; outline `rgb(242,242,242)` (Nimbus) a 100%.
- Botão A pressed: fill muda para `rgb(242,242,242)` (Nimbus) a 95%.
- L3 (left_joy) base: fill `rgb(242,242,242)` (Nimbus) a 50%.
- L3 (left_joy) pressed: `--knob-pressed-opacity` resolve para 0.95 no knob; computed `fill-opacity: 0.95` (verificado com loop `animate()` parado; em produção, o `updateGamepadState(gamepad)` ativa `.pressed` em `left_joy`/`right_joy` quando botões 10/11 são pressionados).
- `g#menu` presente, `g#view1` ausente (renomeação ok).
- 2 trigger overlays (onda LT/RT preservada).
- `window.buboController` inicializado.

### Decisões pendentes

- Nenhuma. O mantenedor confirmou as cores (`Nimbus #F2F2F2 | Umbral #0D0D0D`) e o SVG. As duas decisões técnicas (renomeação `view1`→`menu`, interpretação L3/R3 (a)) foram adotadas pelo agente como defaults não-invariantes e documentadas aqui. Se o mantenedor preferir L3/R3 como elementos separados (interpretação (b)), uma Sprint futura deve adicionar `g#l3` / `g#r3` ao SVG e ajustar o JS.

---

## v1.2610.005 — 2026-10-13 · Redesign outline-only Xbox + L3/R3 knob fix + trigger wave mask + VIEW/MENU icons + hint "no controller"

### Resumo

Sprint grande, com 5 mudanças integradas no Xbox layout, todas derivadas do chat com o mantenedor em 13/10/2026. Arcade layout permanece pendente (o mantenedor entregarará o novo SVG arcade em sprint futura).

### Decisões técnicas (não-invariantes) adotadas pelo agente

- **Renomeação de IDs uppercase → lowercase**: o novo SVG exportado pelo Affinity usa `BUTTON_A`, `LEFT_JOY_KNOB`, `DPAD_UP`, etc. O projeto usa convenção lowercase (`a`, `left_joy`, `up`). O agente renomeou TODOS os IDs no SVG para lowercase antes de inline no `<template id="svg-xbox">` e salvar em `masks/controllers/xbox.svg`. Justificativa: o JS existente (`BUTTON_MAP`, `collectElements()`, `cacheKnobData()`) usa lowercase; renomear no SVG evita retrabalho no JS. Alternativa considerada: adicionar mapeamento uppercase→lowercase no JS. Rejeitada: aumenta acoplamento e adiciona complexidade sem benefício.

- **Ícones VIEW/MENU como `<rect class="button-icon">` em path-local coords**: o agente desenhou os ícones como dois retângulos sobrepostos (View) e três retângulos horizontais (Menu), embutidos como filhos do `<g id="view">` / `<g id="menu">` no SVG template. Posição em path-local coords (não viewBox) para herdar o transform do `<g>` pai. Justificativa: mantém o ícone como parte do SVG (herda transform, posicionamento automático), e o CSS `.button-icon` pinta-os com `var(--accent)` (Nimbus) em idle e `var(--color-bg)` (Umbral) em pressed (mesmo comportamento do label text). Alternativa considerada: adicionar ícones via JS em `addLabels()`. Rejeitada: mistura concerns (labels são texto, ícones são SVG) e adicionaria complexidade ao JS.

- **Trigger wave mask via `clip-path: inset(0 round 9px)`**: o agente usou CSS `clip-path` no `.trigger-overlay` em vez de SVG `<clipPath>` ou canvas `ctx.clip()`. Justificativa: (1) CSS `clip-path` é declarativo, não requer JS extra; (2) funciona em CEF 111+ (invariant §6.8 verificado); (3) o `clip-path` é em coords CSS locais do div (que casam com viewBox units no display 1:1), e o `transform: scale()` no parent `.controller-overlay` escala o conjunto, inclusive o clip — então `?scale=2` funciona corretamente. Alternativa considerada: canvas `ctx.clip()` com `Path2D` do path do trigger. Rejeitada: requereria parse do path data em coords locais e aplicação do transform manualmente — complexidade desnecessária.

- **Hint "no controller" com system font weight 900 (não Sofia Sans XC Black)**: o mantenedor pediu "Sofia Sans XC Black". Invariant §6.3 (offline-first) proíbe carregar fontes externas em runtime. O agente usou a font stack existente (`--label-font`) com peso 900 (`--hint-font-weight: 900`). Justificativa: mantém §6.3 intacto. Limitação documentada no CHANGELOG e ROADMAP (item "Sofia Sans XC Black font embedding" no backlog de Future Technical Improvements). Para matching visual exato, o mantenedor precisa fornecer o `.ttf` para embed como base64 em sprint futura.

- **Hint não reaparece após primeira conexão**: o agente escolheu `gamepadEverConnected` flag que, uma vez `true`, suprime o hint para sempre (mesmo se o gamepad desconectar brevemente). Justificativa: evita ruído durante a live se o controle cair e voltar. Alternativa considerada: reaparecer após N segundos de desconexão. Rejeitada: muito agressiva para uma "TV 24/7"; o hint é primariamente um aviso de "você esqueceu de plugar o controle antes de começar", não um monitor contínuo de conexão.

- **5pt stroke reconciliation**: o mantenedor pediu "traço interno de 5pt (= 5px?)". O agente confirmou via cálculo: em CSS/SVG, `1pt = 1.333px` (porque `1pt = 1/72 inch` e `1px = 1/96 inch`). Então `5pt = 6.667px`. A geometria exportada pelo Affinity mostra gap de `4.5` unidades path-local entre contorno outer e inner. Após o scale `0.666667` do `<g>` pai, isso vira `4.5 × 0.666667 = 3.0` viewBox units. No display 1:1 (640×216 CSS px), o traço é ~3 CSS px — mais fino que os 5pt (6.667px) pedidos. O agente aceitou o SVG como exportado (o mantenedor enviou o arquivo pronto) e documentou a discrepância aqui. Se o mantenedor quiser o traço mais largo, precisará re-exportar do Affinity com stroke maior.

### Mudanças no JS (`processSvg()` reescrito)

- **Antes (v1.2610.004)**: cada `<g>` de botão tinha DOIS filhos (`<circle>` fill + `<path>` anel, ou 2 paths). O JS chamava `replaceRingWithCircle()` para botões circulares (substituía o anel por um `<circle>` sólido) ou `extractOuterSubpath()` para não-circulares (mantinha só o outer subpath).
- **Agora (v1.2610.005)**: cada `<g>` de botão tem UM `<path>` anel (2 subpaths: outer + inner). O JS cria um SEGUNDO `<path>` com apenas o outer subpath (sólido) e insere ANTES do anel. Resultado:
  - `:nth-child(1)` = fill (outer sólido, invisível em idle, cheio em pressed)
  - `:nth-child(2)` = outline (anel, both subpaths, render via `fill-rule: evenodd`)
- **Funções removidas**: `replaceRingWithCircle()`, `extractOuterSubpath()` (não mais necessárias).
- **Skip paths**: root group (`<g id="XBOX">`), `<path id="BACKGROUND">`, reach rings (`g[id$="_reach"]`) — esses não têm fill, só outline (o CSS rule existente para `_reach` cuida deles).

### Bug real capturado pelo Agent Browser

- **Background inicialmente pintado com Nimbus em vez de Umbral**: a regra `.controller-svg g[id] > :nth-child(1) { fill: var(--button-color, var(--accent)) !important; }` (button fill) também casava com o `<path id="BACKGROUND">` (que é `:nth-child(1)` de `<g id="XBOX">`). Sem `!important` na regra do background, a regra do botão vencia e pintava o bg com Nimbus. Corrigido adicionando `!important` à regra `.controller-svg > g > path#BACKGROUND { fill: var(--color-bg) !important; fill-opacity: var(--bg-opacity) !important; }`. Detalhado no comentário CSS acima da regra.

### Verificação Agent Browser

- Zero erros de console, zero erros de página em todos os estados testados.
- Background: `rgb(13, 13, 13)` (Umbral) a 95% ✓
- Botão A idle: fill `rgb(242, 242, 242)` (Nimbus) a 0% (invisível — outline-only) ✓
- Botão A outline: `rgb(242, 242, 242)` (Nimbus) a 100% com `fill-rule: evenodd` (anel visível) ✓
- `<g id="view">` children: 5 (path fill + path outline + 2 rect button-icon + text label) ✓
- `<g id="a">` children: 3 (path fill + path outline + text label "A") ✓
- LT trigger overlay: `clip-path: inset(0px round 9px)` aplicado ✓
- Hint: presente, hidden por default, `.visible` adicionado após 5s sem gamepad (opacity 0.6) ✓
- `?color=xbox`: ABXY colors aplicadas (`--color-a: #4CAF50` etc.) ✓
- `?scale=2`: carrega sem erros ✓
- `?controller=arcade`: regression check — carrega sem erros (arcade SVG não tem rings, `processSvg()` pula) ✓
- VLM visual confirmation de `/tmp/bubo-v005-idle-hint.png`: fundo preto, contornos brancos, "Connect a controller" no centro ✓

### Limitação honesta declarada

- **OBS test matrix (AGENT.md §7.2) NÃO executado pelo agente**: o sandbox não tem gamepad real nem OBS Studio. O agente executou o Agent Browser self-verification (§7.3) e confirmou estado visual + estrutural, mas o mantenedor deve rodar o OBS test matrix completo (botões reais, triggers pressure-sensitive, analog stick) antes de declarar v1.2610.005 production-ready.

### Decisões pendentes

- Nenhuma. Todas as 6 decisões técnicas acima foram adotadas pelo agente como defaults não-invariantes e documentadas. Se o mantenedor preferir alguma alternativa, uma Sprint futura pode ajustar.

---

## v1.2610.006 — 2026-10-13 · Bugfixes pós-teste OBS (3 bugs reportados pelo mantenedor)

### Resumo

O mantenedor testou v1.2610.005 no OBS e reportou 3 bugs. Esta sprint é puramente de correção, sem novas features.

### Bug 1 — Ícones VIEW/MENU apareciam por padrão

**Sintoma**: os ícones (dois quadrados para View, hambúrguer para Menu) estavam visíveis sem qualquer argumento na URL.

**Causa-raiz**: em v1.2610.005, os `<rect class="button-icon">` foram embutidos no SVG template e a CSS não tinha regra `display` — então eles renderizavam por padrão. O mantenedor queria opt-in.

**Correção**: CSS `.button-icon { display: none; }` por padrão. Nova classe `.show-viewmenu-icons` no overlay unhides via `.controller-overlay.show-viewmenu-icons .controller-svg g[id] .button-icon { display: inline; }`. JS `applyViewMenuIcons()` lê `?viewmenu=` (aceita `1`, `true`, `yes`, `on`) e adiciona a classe ao overlay.

**Decisão técnica**: parâmetro `?viewmenu=` separado (não estende `?icons=`). Razão: `?icons=` controla TEXT labels para ABXY/LB/RB/LT/RT/L3/R3. Ícones VIEW/MENU são conceito separado (formas SVG embutidas, não texto). Acoplar forçaria o usuário a ativar um set de labels de texto só para mostrar os ícones. Desacoplar é mais limpo.

### Bug 2 — JOY_REACH invisível

**Sintoma**: o círculo maior de contorno (reach boundary) ao redor de cada knob analógico não aparecia.

**Causa-raiz (dupla)**:
1. **CSS antigo incompatível com SVG novo**: o SVG novo (v1.2610.005) desenha cada reach group (`left_joy_reach`, `right_joy_reach`) com **UM** `<path>` (anel: outer + inner subpaths). O CSS de v1.2610.004 tinha `:nth-child(1) { fill: none }` que no SVG antigo (2 paths: disco sólido + anel) escondia o disco e mostrava o anel. No SVG novo, o único path É `:nth-child(1)`, então `fill: none` o escondia inteiramente.
2. **Guerra de especificidade CSS**: mesmo removendo `fill: none`, a regra de button-fill `.controller-svg g[id] > :nth-child(1) { fill-opacity: var(--button-fill-opacity); fill-rule: nonzero !important; }` (especificidade 0,0,3,1) vencia sobre a regra de reach `.controller-svg g[id$="_reach"] > path` (especificidade 0,0,2,2). O reach path herdava `fill-opacity: 0` (button fill idle) e `fill-rule: nonzero` (disco sólido em vez de anel) — invisível.

**Correção**: regra de reach reescrita com **IDs explícitos**: `g#left_joy_reach > path, g#right_joy_reach > path, g#joy_reach > path`. IDs têm especificidade (0,1,0,0) que vence o seletor de atributo (0,0,3,1) no segundo slot. Adicionado `!important` em `fill-opacity: 1` e `fill-rule: evenodd`. Reach agora renderiza como anel visível (Nimbus, 100% opaco) ao redor de cada knob.

**Lições de especificidade CSS**: quando uma regra genérica (seletor de atributo `g[id$="_reach"]`) conflita com uma regra específica (seletor de ID `g#left_joy_reach`), o ID vence independente de `!important` tie-breaking. Sempre que uma regra "deve sempre vencer", use ID explícito — não confie em `!important` sozinho contra seletores genéricos.

### Bug 3 — `?color=xbox` não coloria o contorno

**Sintoma**: com `?color=xbox` ativo, os contornos dos botões ABXY estavam todos brancos (Nimbus). As cores (verde/vermelho/azul/amarelo) não apareciam.

**Causa-raiz**: em v1.2610.005, a CSS do outline foi mudada para `fill: var(--accent) !important` (sempre Nimbus), quebrando o comportamento de v1.2610.004 onde o outline usava `var(--button-color)` (que resolve para `--color-a/b/x/y` para ABXY buttons, com fallback `--accent` = Nimbus quando nenhum `?color=` está ativo).

**Correção**: CSS do outline revertida para `fill: var(--button-color, var(--accent)) !important`. Agora:
- Default (sem `?color=`): outline = Nimbus (porque `--color-a/b/x/y` default = `var(--accent)` = Nimbus).
- `?color=xbox`: outline = verde A, vermelho B, azul X, amarelo Y.
- `?color=playstation`: outline = azul Cross, vermelho Circle, rosa Square, verde Triangle.
- O pressed fill continua usando o mesmo `var(--button-color, var(--accent))` (já correto em v1.2610.005) a 100% opacidade (`--button-pressed-opacity: 1`), então as cores ABXY aparecem TANTO no contorno (idle) QUANTO no preenchimento (pressed).

**Confirmação do mantenedor**: "O contorno poderia continuar colorido, como era e a cor de preenchimento deve ser 100% opaca, não deve ter transparência." A correção atende ambos: contorno colorido + pressed fill 100% opaco.

### Verificação Agent Browser (via file://)

Servidor HTTP localhost morria quando agent-browser abria sessão (provável issue de lifecycle do processo). Solução: abrir via `file:///home/z/my-project/repos/bubo-controller/index.html` (mesmo protocolo que o OBS usa para Browser Source local). Funcionou estavelmente.

- **Default state**: reach `rgb(242,242,242)` (Nimbus) com `fill-rule: evenodd`, `fill-opacity: 1` ✓; outline A `rgb(242,242,242)` (Nimbus) ✓; view icon `display: none` ✓; overlay class `controller-overlay controller-xbox` (sem `show-viewmenu-icons`) ✓.
- **`?color=xbox`**: outline A `rgb(76,175,80)` (verde), B `rgb(229,52,42)` (vermelho), X `rgb(14,125,192)` (azul), Y `rgb(242,194,0)` (amarelo) ✓.
- **`?viewmenu=1`**: overlay class `controller-overlay show-viewmenu-icons controller-xbox` ✓; view icon `display: inline`, `fill: rgb(242,242,242)` (Nimbus) ✓; menu icon `display: inline` ✓.
- **`?controller=arcade`** (regressão): `joy_reach` path `fill: rgb(242,242,242)`, `fill-rule: evenodd`, `fill-opacity: 1` ✓ (visível, sem regressão).
- **VLM confirmação visual** de `/tmp/bubo-v006-color-xbox.png`: "botão A verde, B vermelho, X azul, Y amarelo" (cores ABXY no contorno); "analógicos têm um círculo maior de contorno visível" (reach visível); "VIEW e MENU estão vazios, sem ícones visíveis" (ícones ocultos por padrão) ✓.
- **VLM confirmação visual** de `/tmp/bubo-v006-color-viewmenu.png`: "Botão VIEW (esquerda): dois quadrados sobrepostos. Botão MENU (direita): três linhas horizontais (hambúrguer)" ✓ (ícones visíveis com `?viewmenu=1`).

### Limitação honesta declarada

- **OBS test matrix (AGENT.md §7.2) NÃO executado pelo agente** — sandbox sem gamepad/OBS. O mantenedor já testou v1.2610.005 no OBS (foi como descobriu os bugs); esta sprint corrige os bugs reportados. O mantenedor deve re-testar v1.2610.006 no OBS (pull + reload da Browser Source) para confirmar que os 3 fixes resolvem no ambiente real.

---

## v1.2610.007 — 2026-10-13 · Ajuste de posicionamento dos botões do topo

### Resumo

Sprint simples, puramente de atualização de SVG com ajuste de posicionamento dos 6 botões do topo (VIEW, MENU, LB, LT, RB, RT). Sem mudanças em JS ou CSS — só substituição do arquivo SVG e do template inline.

### Mudança geométrica

O mantenedor enviou novo `xbox.svg` com os 6 botões do topo deslocados +18 unidades de viewBox para baixo (Y translate `9` → `26.999982`). Isso adiciona respiro entre esses botões e a borda superior arredondada do controle, endereçando aperto visual que o mantenedor observou em teste no OBS.

**Antes (v1.2610.005/006)**:
- view/menu/lb/lt/rb/rt todos com Y translate = 9
- Centers calculados (via getCTM): todos em cy=27

**Depois (v1.2610.007)**:
- view/menu/lb/lt/rb/rt todos com Y translate = 26.999982
- Centers calculados (via getCTM): todos em cy=45

**Não mudou**: ABXY (a/b/x/y), D-pad (up/down/left/right), analógicos (left_joy/right_joy + reach groups). Todos mantêm os mesmos transforms da v1.2610.005.

### Limpeza cosmética nos transforms do D-pad

O SVG novo normalizou `-0` para `0` nas matrizes dos grupos `down` e `left`:
- `down`: `matrix(-0.222222,0,-0,-0.333333,301.111111,207)` → `matrix(-0.222222,0,0,-0.333333,301.111111,207)`
- `left`: `matrix(-0,-0.222222,0.333333,-0,176,188.111111)` → `matrix(0,-0.222222,0.333333,0,176,188.111111)`

Matematicamente equivalente (`-0 === 0` em IEEE 754). Nenhuma mudança visual.

### Re-embutição dos ícones VIEW/MENU

O SVG do mantenedor vem com apenas os anéis (paths) dos botões — não inclui os rects `.button-icon` que adicionei em v1.2610.005 para os ícones de VIEW (dois quadrados) e MENU (hambúrguer). Política do agente: **re-embutir os rects em cada atualização de SVG do mantenedor** para que os ícones sobrevivam entre refreshes de SVG.

**Por que não estão no SVG original do mantenedor**: o mantenedor exporta as formas geométricas "físicas" do controle do Affinity. Os ícones (metáforas UI: dois-quadrados, hambúrguer) são uma camada UX de runtime que o agente adiciona — não são parte da geometria "física" do controle. Esta separação está documentada em `masks/controllers/README.md` (o SVG é source-of-truth da geometria do controle; ícones são adição de runtime gerenciada pelo agente).

**Como os rects se movem automaticamente**: os rects estão em coords path-local (0-320 no X, 0-54 no Y, que é o espaço do path). Eles são filhos do `<g>` que tem o transform. Quando o `<g>` se move, os rects se movem junto automaticamente — não preciso atualizar as coords dos rects manualmente. Isso é a vantagem de manter ícones em path-local coords em vez de viewBox coords.

### Verificação Agent Browser (via file://)

Servidor HTTP localhost tinha morrido em sprints anteriores ao rodar agent-browser; usei `file:///home/z/my-project/repos/bubo-controller/index.html` (mesma forma que o OBS abre Browser Source local). Funcionou estavelmente.

**Posicionamentos (via `getBBox()` + `getCTM()` matrix transform)**:

| Botão | Center (x, y) v1.2610.005 | Center (x, y) v1.2610.007 | Mudança |
|---|---|---|---|
| view | (302, 27) | (302, 45) | +18 Y ✓ |
| menu | (338, 27) | (338, 45) | +18 Y ✓ |
| lb | (230, 27) | (230, 45) | +18 Y ✓ |
| lt | (104, 27) | (104, 45) | +18 Y ✓ |
| rb | (410, 27) | (410, 45) | +18 Y ✓ |
| rt | (536, 27) | (536, 45) | +18 Y ✓ |
| up | (248, 108) | (248, 108) | unchanged ✓ |
| down | (248, 162) | (248, 162) | unchanged ✓ |
| left | (221, 135) | (221, 135) | unchanged ✓ |
| right | (275, 135) | (275, 135) | unchanged ✓ |
| a | (392, 171) | (392, 171) | unchanged ✓ |
| b | (428, 135) | (428, 135) | unchanged ✓ |
| x | (356, 135) | (356, 135) | unchanged ✓ |
| y | (392, 99) | (392, 99) | unchanged ✓ |
| left_joy | (104, 135) | (104, 135) | unchanged ✓ |
| right_joy | (536, 135) | (536, 135) | unchanged ✓ |

**Regressões**:
- `?color=xbox`: outline A `rgb(76,175,80)` (verde) ✓ (v1.2610.006 fix preservado)
- `?viewmenu=1`: overlay class `controller-overlay show-viewmenu-icons controller-xbox`, view icon `display: inline` ✓ (v1.2610.006 fix preservado)
- Zero erros no console ✓

**VLM confirmação visual** de `/tmp/bubo-v007-default.png`:
- "os 6 elementos do topo estão alinhados em uma linha horizontal perto da parte superior do controle"
- "ABXY e analógicos estão mais abaixo, na região central e inferior"
- "há um espaço visível (margem/respiro) entre a linha dos botões superiores e a borda superior arredondada preta do controle. Eles não estão colados na borda."

✓ Match exato com a intenção do mantenedor.

### Decisões técnicas não-invariantes

Nenhuma nova decisão. A sprint seguiu o padrão estabelecido em v1.2610.005/006:
- IDs lowercase (mantenedor já seguiu a convenção desta vez)
- Ícones embutidos em path-local coords (re-embutidos automaticamente)
- CSS e JS unchanged (regras de button-fill/outline/reach já abrangem o novo SVG)
- Verificação via `file://` (mais estável que HTTP localhost para agent-browser)

### Limitação honesta

OBS test matrix (AGENT.md §7.2) não executado pelo agente (sandbox sem gamepad/OBS). Mantenedor deve re-testar v1.2610.007 no OBS (pull + reload da Browser Source) para confirmar o novo posicionamento no ambiente real.

### Decisões pendentes

Nenhuma.
