# AUDIT.md

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
