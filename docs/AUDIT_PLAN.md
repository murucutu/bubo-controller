# AUDIT_PLAN.md

> **Nota histórica (v1.2610.002):** Este é o plano de auditoria **original** que conduziu à consolidação do projeto em `index.html` (Track A/B, baseada em `masks/bg_dark.png`). O projeto evoluiu desde então para **Track C** (SVG inline, sem PNG) — ver [VARIANTS.md](../VARIANTS.md) e [CHANGELOG.md](../CHANGELOG.md) `v1.2610.001`. Este arquivo é preservado como registro do planejamento histórico; as referências a `bg_dark.png` e ao pipeline Python descrevem o estado **anterior**, não o atual.

## Objetivo revisado

Eliminar excesso do projeto e transformar o overlay em um único arquivo HTML autocontido para OBS:

- `index.html` será o novo arquivo principal, baseado no leiaute novo de `index2.html`.
- `index2.html` deixará de existir depois da migração.
- `style.css` e `controller.js` serão removidos se não houver dependências reais.
- Scripts Python e arquivos de geração/posição serão removidos se não forem necessários para o runtime OBS.
- O problema visual dos analógicos LS/RS será corrigido no novo `index.html`.

## Respostas de planejamento

### Precisamos de scripts Python?

Não para o overlay final de OBS.

O plano atual é remover da árvore principal:

- `analyze_masks.py`
- `generate_overlay_uv.py`
- `generate_overlay.py`
- `generate_css.py`
- `positions.json`

Esses arquivos serão removidos apenas após confirmação, via PowerShell, de que não existem referências necessárias no projeto.

### Precisamos de HTML/CSS/JS separados?

Não para o caso de uso de overlay OBS.

O plano atual é consolidar tudo em um único `index.html` contendo:

- HTML estrutural;
- CSS inline;
- JavaScript inline;
- nenhuma dependência de `style.css` ou `controller.js`.

Isso reduz atrito no OBS Browser Source e elimina divergência entre arquivos separados.

## Escopo refinado

### Arquivos mantidos

- `index.html`
  - Novo arquivo principal após substituição por `index2.html`.
- `masks/bg_dark.png`
  - Imagem usada pelo background do overlay.
- `masks/`
  - Diretório mantido apenas se contiver assets usados pelo novo overlay.

### Arquivos candidatos à remoção

- `index2.html`
  - Fonte temporária do novo overlay; removido após migrar para `index.html`.
- `style.css`
  - Removido após consolidar CSS inline.
- `controller.js`
  - Removido após consolidar JS inline.
- `analyze_masks.py`
  - Removido se não houver necessidade de análise de máscaras no runtime.
- `generate_overlay_uv.py`
  - Removido se não houver pipeline de geração necessário.
- `generate_overlay.py`
  - Removido como gerador legado.
- `generate_css.py`
  - Removido como gerador legado.
- `positions.json`
  - Removido se o novo overlay não depender mais de geração baseada em posições.
- `README.md`
  - Removido se for documentação obsoleta para o fluxo antigo.

### Arquivos de máscara candidatos à remoção

Manter apenas assets referenciados pelo novo overlay.

Atualmente o novo overlay usa diretamente:

- `masks/bg_dark.png`

Candidatos prováveis à remoção:

- `masks/a.png`
- `masks/b.png`
- `masks/x.png`
- `masks/y.png`
- `masks/lt.png`
- `masks/rt.png`
- `masks/lb.png`
- `masks/rb.png`
- `masks/d-up.png`
- `masks/d-down.png`
- `masks/d-left.png`
- `masks/d-right.png`
- `masks/ls.png`
- `masks/rs.png`
- `masks/view.png`
- `masks/menu.png`
- `masks/bg.png`
- `masks/masks.af`

## Diagnóstico refinado

1. O problema ocorre no estado neutro, antes de qualquer entrada de gamepad/WebHID. Portanto a causa não é conexão, estado ligado ou leitura de eixo.
2. Em `index2.html`, a área visual do analógico deve seguir as máscaras:
   - `ls.png`: centro base `1632, 1944`, tamanho `172x172`.
   - `rs.png`: centro base `2208, 1944`, tamanho `172x172`.
3. Com o crop definido em `--crop-x: 1536px` e `--crop-y: 1728px`, as posições relativas corretas são:
   - LS: `left: 10px; top: 120px; width: 172px; height: 172px;`.
   - RS: `left: 586px; top: 120px; width: 172px; height: 172px;`.
4. O CSS atual usa boundary `192x192`:
   - LS: `left: 0; top: 120px; width: 192px; height: 192px;`.
   - RS: `left: 576px; top: 120px; width: 192px; height: 192px;`.
5. O knob atual usa pixels hardcoded derivados de uma geometria inconsistente:
   - `width: 114.6667px; height: 114.6667px;`.
   - `left: 38.6667px; top: 38.6667px;`.
6. O JavaScript sobrescreve `transform` inline em `updateAnalogStick()`. Isso impede que a centralização visual seja mantida exclusivamente por CSS. A correção deve separar:
   - centralização estática do knob no boundary;
   - deslocamento dinâmico do stick por variáveis CSS.

## Plano de execução após aprovação explícita

### 1. Auditoria de referências com PowerShell

Executar comandos PowerShell para confirmar dependências antes de remover arquivos:

```powershell
Get-ChildItem -File -Recurse | Select-String -Pattern "index2\.html|style\.css|controller\.js|generate_|analyze_masks|positions\.json|masks/"
Select-String -Path .\index2.html -Pattern "masks/|style\.css|controller\.js|positions|generate_|analyze_masks" -AllMatches
```

### 2. Limpeza de excesso com PowerShell

Após confirmar que os arquivos são excedentes:

```powershell
Remove-Item -LiteralPath .\index2.html,.\style.css,.\controller.js,.\analyze_masks.py,.\generate_overlay_uv.py,.\generate_overlay.py,.\generate_css.py,.\positions.json,.\README.md -ErrorAction SilentlyContinue
Remove-Item -LiteralPath .\masks\a.png,.\masks\b.png,.\masks\x.png,.\masks\y.png,.\masks\lt.png,.\masks\rt.png,.\masks\lb.png,.\masks\rb.png,.\masks\d-up.png,.\masks\d-down.png,.\masks\d-left.png,.\masks\d-right.png,.\masks\ls.png,.\masks\rs.png,.\masks\view.png,.\masks\menu.png,.\masks\bg.png,.\masks\masks.af -ErrorAction SilentlyContinue
```

### 3. Migração do novo overlay

- Copiar `index2.html` para `index.html`.
- Aplicar no novo `index.html` a correção de centralização dos analógicos.
- Remover `index2.html` após a migração.
- Garantir que `index.html` seja autocontido, sem CSS/JS externos.

### 4. Correção dos analógicos

No novo `index.html`:

1. Corrigir o boundary dos analógicos para o tamanho real da máscara:
   - LS: `left: 10px; top: 120px; width: 172px; height: 172px;`.
   - RS: `left: 586px; top: 120px; width: 172px; height: 172px;`.
2. Centralizar o knob por CSS:
   - `left: 50%; top: 50%; width: 66.6667%; height: 66.6667%;`.
   - `transform: translate(calc(-50% + var(--stick-x, 0px)), calc(-50% + var(--stick-y, 0px)))`.
3. Alterar `updateAnalogStick()` para não sobrescrever a centralização:
   - Definir `--stick-x` e `--stick-y`.
   - Não definir `knob.style.transform`.
4. Garantir estado neutro explícito:
   - No carregamento e quando não houver gamepad, `--stick-x: 0px` e `--stick-y: 0px`.

### 5. Validação

- Abrir o novo `index.html` sem controle conectado e confirmar LS/RS centralizados no estado neutro.
- Mover analógicos com gamepad conectado e confirmar deslocamento visual preservado.
- Confirmar que o OBS Browser Source carrega um único arquivo HTML.
- Confirmar que não existem referências pendentes a `index2.html`, `style.css`, `controller.js`, scripts Python removidos ou arquivos de máscara removidos.

## Estrutura obrigatória do AUDIT.md final

### 1. Otimizações e Remoção de Código-Lixo

- Identificar duplicação de regras de analógico no CSS inline.
- Apontar pixels hardcoded duplicados para tamanho/posição do knob.
- Documentar remoção de arquivos excedentes: CSS/JS externos, scripts Python, geradores, posição JSON, README obsoleto e máscaras não usadas.
- Propor centralização por `left/top: 50%` + variáveis CSS para reduzir complexidade e evitar divergência entre CSS e JS.

### 2. Robustez, Tratamento de Erros e Prevenção

- Avaliar sobrescrita de `transform` pelo JavaScript.
- Verificar reset de estado neutro quando não há gamepad.
- Identificar ruído de eixo, dead zone e atualização contínua por `requestAnimationFrame`.
- Avaliar listeners, contexts de canvas e desenho dos canvases de gatilho.

### 3. SOTA (Estado da Arte), Tipagem e Padronização

- Auditoria PEP 8 apenas se scripts Python forem mantidos; caso sejam removidos, documentar que deixam de ser parte do runtime.
- Verificar type hints em funções/métodos Python apenas se scripts Python forem mantidos.
- Propor centralização de constantes visuais e padronização de nomes no HTML consolidado.

### 4. Lazy Processing e Alocação de Recursos

- Revisar `requestAnimationFrame` e desenho dos canvases de gatilho.
- Identificar cálculos/desenhos desnecessários quando não há controle conectado.
- Avaliar listeners e contexts de canvas alocados no construtor.

### 5. SSOT (Single Source of Truth) e Hardcoding

- Garantir que posições, unidades, dead zones e offsets venham de fonte única dentro do HTML consolidado.
- Listar números mágicos e strings hardcoded relevantes no novo `index.html`.
- Documentar remoção de `positions.json` e geradores Python como simplificação do projeto.

### 6. Roadmap de Evolução

- Curto prazo: corrigir boundary e centralização dos analógicos no novo `index.html`.
- Médio prazo: centralizar constantes visuais dentro do HTML ou em um futuro arquivo de configuração se o projeto voltar a crescer.
- Longo prazo: testes visuais automatizados para OBS, tipagem estática se JS for separado futuramente e regeneração opcional de assets sem reintroduzir complexidade no runtime.

## Regra da fase atual

Esta fase é somente planejamento. Nenhum arquivo de código-fonte ou asset existente deve ser alterado antes da autorização explícita.
