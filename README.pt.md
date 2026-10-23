# Bubo Controller

**Um overlay de controle com olhos de coruja para o OBS Browser Source.**

[![License: Unlicense](https://img.shields.io/badge/licença-Unlicense-blue.svg)](LICENSE)
[![Feito para OBS](https://img.shields.io/badge/feito%20para-OBS-9146FF.svg)](https://obsproject.com/)
[![Idiomas: EN | PT-BR](https://img.shields.io/badge/idiomas-EN%20%7C%20PT--BR-success.svg)](#idiomas)
![Versão](https://img.shields.io/badge/version-v1.2610.008-9146FF.svg)

**Versão:** `v1.2610.008` · **Esquema:** `v1.YYMM.XXX` (v1 = publicado · YYMM = mês da build · XXX = número da Sprint, reseta mensalmente) — veja [CHANGELOG.md](CHANGELOG.md).

**Idiomas:** [English](README.md) · Português (BR)

---

## Sobre

Bubo Controller é um overlay em arquivo HTML único que lê a [Gamepad API](https://developer.mozilla.org/pt-BR/docs/Web/API/Gamepad_API) do navegador e desenha um controle na sua cena do OBS em tempo real. Suporta múltiplos layouts de controle (Xbox, Arcade fightstick) via blueprints SVG trocáveis. Gatilhos são sensíveis à pressão, analógicos são calibrados, e cada botão tem cor individualmente customizável.

**Por que "Bubo"?** *Bubo* é o nome em latim para a coruja-orelhuda, e era a coruja que acompanhava Minerva, deusa romana da sabedoria. A coruja observa os streamers desde a antiguidade — agora observa o seu controle. 🦉

O projeto é dedicado ao **domínio público** ([The Unlicense](LICENSE)) — use, faça fork, venda, derive. Sem necessidade de atribuição.

---

## Características

- ✅ Single-file HTML — sem build, sem dependências, sem requests externos
- ✅ **Múltiplos layouts de controle** via blueprints SVG (`?controller=xbox|arcade`)
- ✅ Gamepad API padrão (funciona com Xbox, PlayStation, controles genéricos)
- ✅ Gatilhos LT/RT analógicos com gráfico de onda reativo (layout Xbox)
- ✅ Analógicos calibrados com dead zone
- ✅ **Layout Arcade** com rastro ghost-glow do stick (1/3 do knob, captura analógico + d-pad)
- ✅ Cores por botão via variáveis CSS (`--color-a`, `--color-b`, …)
- ✅ **Paletas de cores ABXY** via URL (`?color=xbox|playstation`)
- ✅ Labels de botões trocáveis via URL (`?icons=xbox|playstation|nintendo|none`)
- ✅ **Sistema de opacidade em camadas** — 40% fundo + 40% botões = ~64% combinado; 100% ao pressionar
- ✅ Auto-escala para qualquer tamanho de Browser Source mantendo a proporção
- ✅ Modos de ajuste opcionais (`?fit=contain|cover`)
- ✅ Modo debug (duplo clique no overlay) para inspeção visual
- ✅ Arte baseada em SVG (arquitetura Trilha C — SVG inline, animado por JS)

---

## Início rápido

1. **Baixe e descompacte** o projeto (ou `git clone https://github.com/murucutu/bubo-controller.git`).
2. No OBS Studio, adicione uma fonte **Navegador** (Browser).
3. Marque **Arquivo local** e selecione `index.html`.
4. Defina **Largura** e **Altura**. Para full-bleed (sem padding transparente), use qualquer tamanho ~2.96:1 (o canvas Xbox do v1.2610.004 é 640×216):
   - `640 × 216` (base, escala 1×)
   - `1280 × 432` (2× — HD horizontal full-bleed)
   - `1920 × 648` (3× — Full HD horizontal full-bleed)
   - `3840 × 1296` (6× — 4K horizontal full-bleed)
   - Ou qualquer 16:9 (`1920×1080`, `3840×2160`) — o overlay preenche a largura, com padding vertical.
5. Conecte um controle. Pressione qualquer botão. O overlay reage em tempo real.

> Dica: ative **"Fechar fonte quando não estiver visível"** para economizar recursos quando a cena não estiver ativa.

---

## Parâmetros de URL

| Parâmetro | Valores | Descrição |
| --------- | ------- | ----------- |
| `?controller=` | `xbox` (padrão) / `arcade` | Layout de controle a exibir |
| `?color=` | `xbox` / `playstation` | Esquema de cores ABXY — colorea TANTO o contorno QUANTO o preenchimento pressionado (padrão: tudo Nimbus/branco) |
| `?icons=` | `xbox` (padrão) / `playstation` / `nintendo` / `none` | Set de labels de texto dos botões (ABXY, LB/RB, LT/RT, L3/R3) |
| `?viewmenu=` | `1` / `true` / `yes` / `on` | Mostra os ícones VIEW/MENU (dois quadrados para View, hambúrguer para Menu). Ocultos por padrão. |
| `?scale=X` | qualquer número positivo | Override manual de escala (ex.: `?scale=2` = 2× zoom) |
| `?fit=` | `contain` (padrão) / `cover` | Como o overlay ajusta à source: contain = mantém aspect com padding; cover = preenche a source, pode clipar os lados |

Combine livremente: `?controller=xbox&color=xbox&viewmenu=1&scale=2`

---

## Personalização

Toda a configuração está em `:root` no topo do bloco `<style>` dentro de `index.html`. Sem build.

```css
:root {
    /* Paleta padrão v1.2610.004 */
    --nimbus: #F2F2F2;        /* cor do contorno (botões, knob L3/R3) — cinza claro */
    --umbral: #0D0D0D;        /* cor do preenchimento (fundo, botões) — quase preto */
    --accent: var(--nimbus);  /* alias (contornos) */
    --color-bg: var(--umbral);/* alias (preenchimento + contraste do label) */

    /* Cores ABXY (definidas por ?color= ou manualmente) */
    --color-a: var(--accent);
    --color-b: var(--accent);
    --color-x: var(--accent);
    --color-y: var(--accent);

    /* L3/R3 knob = Nimbus mesmo com ?color=xbox (não segue o esquema ABXY) */
    --color-l3: var(--nimbus);
    --color-r3: var(--nimbus);

    /* Sistema de opacidade (v1.2610.004) */
    --bg-opacity: 0.2;              /* opacidade do fundo (20%) */
    --button-fill-opacity: 0.95;    /* opacidade do botão não pressionado (95%) */
    --button-pressed-opacity: 0.95; /* opacidade do botão pressionado (95%) */
    --knob-fill-opacity: 0.5;       /* base do knob L3/R3 (50% Nimbus) */
    --knob-pressed-opacity: 0.95;  /* knob L3/R3 pressionado (95% Nimbus) */

    /* Rastro do stick (Arcade apenas) */
    --stick-trail-length: 30;       /* número de frames no rastro */
    --stick-trail-fade: 0.04;       /* taxa de fade por frame (menor = mais suave) */
    --stick-trail-radius: 12;       /* raio do círculo do rastro (1/3 do knob) */
    --stick-trail-blur: 12px;       /* blur do ghost glow */
}
```

Veja **[CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md)** para paletas completas, ajuste de opacidade, ajuste do rastro do stick, e como criar SVGs de controle personalizados.

> ℹ️ As documentações detalhadas estão em **inglês** para alcance internacional. Este README cobre o essencial em PT-BR.

---

## Estrutura do projeto

```
.
├── index.html                  # Overlay unificado (Xbox + Arcade, SVG inline)
├── masks/
│   ├── controllers/
│   │   ├── xbox.svg            # Blueprint SVG do controle Xbox (fonte da verdade)
│   │   ├── arcade.svg          # Blueprint SVG do fightstick arcade (fonte da verdade)
│   │   └── README.md           # Como criar/editar SVGs de controle
│   ├── README.md               # Guia de assets customizados
│   └── labels/
│       ├── sprite_example.svg  # Modelo de sprite pack SVG (Fase 5)
│       └── README.md           # Como criar sprite packs
├── AGENT.md                    # Manual operacional do Senior PM (versionamento, invariantes, DoD)
├── CHANGELOG.md                # Histórico versionado (com trechos das conversas)
├── VERSION                     # Versão atual (v1.YYMM.XXX)
├── CUSTOMIZATION_GUIDE.md      # Cores, opacidade, rastro do stick, labels
├── DESIGN_GUIDE.md             # Grid, proporções, timing
├── VARIANTS.md                 # Workflow SVG para novos layouts de controle
├── LABEL_PACKS_GUIDE.md        # Sprite sheets SVG
├── ROADMAP.md                  # Roadmap público
├── CONTRIBUTING.md             # Como contribuir
├── README.md                   # Versão em inglês (primária)
├── README.pt.md                # Este arquivo (PT-BR)
├── LICENSE                     # The Unlicense (domínio público)
├── .gitignore
└── docs/
    ├── AUDIT.md                # Histórico de auditoria (PT-BR)
    └── AUDIT_PLAN.md           # Notas de planejamento (PT-BR)
```

---

## Licença

**[The Unlicense](LICENSE)** — dedicação ao domínio público. Você pode usar, copiar, modificar, publicar, distribuir, sublicenciar e vender este software sem qualquer restrição. Sem necessidade de atribuição.
