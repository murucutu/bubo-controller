# Bubo Controller

**Um overlay de controle com olhos de coruja para o OBS Browser Source.**

[![License: Unlicense](https://img.shields.io/badge/licença-Unlicense-blue.svg)](LICENSE)
[![Feito para OBS](https://img.shields.io/badge/feito%20para-OBS-9146FF.svg)](https://obsproject.com/)
[![Idiomas: EN | PT-BR](https://img.shields.io/badge/idiomas-EN%20%7C%20PT--BR-success.svg)](#idiomas)

**Idiomas:** [English](README.md) · Português (BR)

---

## Sobre

Bubo Controller é um overlay em arquivo HTML único que lê a [Gamepad API](https://developer.mozilla.org/pt-BR/docs/Web/API/Gamepad_API) do navegador e desenha um controle estilo Xbox na sua cena do OBS em tempo real. Gatilhos são sensíveis à pressão, analógicos são calibrados, e cada botão tem cor individualmente customizável.

**Por que "Bubo"?** *Bubo* é o nome em latim para a coruja-orelhuda, e era a coruja que acompanhava Minerva, deusa romana da sabedoria. A coruja observa os streamers desde a antiguidade — agora observa o seu controle. 🦉

O projeto é dedicado ao **domínio público** ([The Unlicense](LICENSE)) — use, faça fork, venda, derive. Sem necessidade de atribuição.

---

## Características

- ✅ Single-file HTML — sem build, sem dependências, sem requests externos (exceto `masks/bg_dark.png`)
- ✅ Gamepad API padrão (funciona com Xbox, PlayStation, controles genéricos)
- ✅ Gatilhos LT/RT analógicos com gráfico de onda reativo à pressão
- ✅ Analógicos LS/RS calibrados com dead zone
- ✅ Cores por botão via variáveis CSS (`--color-a`, `--color-b`, …)
- ✅ Labels de botões trocáveis via URL (`?icons=xbox|playstation|nintendo|none`)
- ✅ Auto-escala para qualquer tamanho de Browser Source mantendo a proporção
- ✅ Modos de ajuste opcionais (`?fit=contain|cover`)
- ✅ Modo debug (duplo clique no overlay) para inspeção visual
- ✅ Acessibilidade: `aria-label` em elementos interativos, `aria-hidden` em decorativos

---

## Início rápido

1. **Baixe e descompacte** o projeto (ou `git clone https://github.com/murucutu/bubo-controller.git`).
2. No OBS Studio, adicione uma fonte **Navegador** (Browser).
3. Marque **Arquivo local** e selecione `index.html`.
4. Defina **Largura** e **Altura**. Para full-bleed (sem padding transparente), use qualquer tamanho ~2.37:1:
   - `768 × 324` (base, escala 1×)
   - `1920 × 810` (2.5× — HD full-bleed horizontal)
   - `3840 × 1620` (5× — 4K full-bleed horizontal)
   - Ou qualquer 16:9 (`1920×1080`, `3840×2160`) — o overlay preenche a largura, com padding vertical.
5. Conecte um controle. Pressione qualquer botão. O overlay reage em tempo real.

> Dica: ative **"Fechar fonte quando não estiver visível"** para economizar recursos quando a cena não estiver ativa.

---

## Parâmetros de URL

| Parâmetro | Valores | Descrição |
| --------- | ------- | ----------- |
| `?scale=X` | qualquer número positivo | Override manual de escala (ex.: `?scale=2` = 2× zoom) |
| `?fit=` | `contain` (padrão) / `cover` | Como o overlay ajusta à source: contain = mantém aspect com padding; cover = preenche a source, pode clipar os lados |
| `?icons=` | `xbox` (padrão) / `playstation` / `nintendo` / `none` | Set de labels dos botões |

Combine livremente: `?fit=cover&icons=playstation&scale=2`

---

## Personalização

Toda a configuração está em `:root` no topo do bloco `<style>` dentro de `index.html`. Sem build.

```css
:root {
    --accent: #FFFFFF;        /* cor de fallback para qualquer botão */
    --color-bg: #000000;      /* referência de cor de fundo */

    /* Cores por botão (default = --accent) */
    --color-a: var(--accent);
    --color-b: var(--accent);
    --color-x: var(--accent);
    --color-y: var(--accent);
    /* ...etc */

    --label-a: "A";          /* label de texto por botão */
    --label-b: "B";
    /* ...etc */

    --stroke: 5px;            /* espessura dos traços */
    --opacity: 0.80;          /* opacidade geral do overlay */
}
```

Para deixar o botão A vermelho:

```css
:root { --color-a: #FF0000; }
```

Veja **[CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md)** para paletas completas, formas de botões, fundos personalizados e sets de labels.

> ℹ️ As documentações detalhadas (`CUSTOMIZATION_GUIDE.md`, `DESIGN_GUIDE.md`, etc.) estão em **inglês** para alcance internacional. O `README.pt.md` (este arquivo) cobre o essencial. Se precisar de tradução de um guia específico, abra uma issue.

---

## Documentação

| Arquivo | Para quem | O que cobre |
| ------- | --------- | ------------ |
| [CUSTOMIZATION_GUIDE.md](CUSTOMIZATION_GUIDE.md) | Usuários | Cores, formas, labels, fundos personalizados (em inglês) |
| [DESIGN_GUIDE.md](DESIGN_GUIDE.md) | Designers | Grid, proporções, espaçamentos, z-index, timing (em inglês) |
| [LABEL_PACKS_GUIDE.md](LABEL_PACKS_GUIDE.md) | Avançado | Sprite sheets SVG para packs de ícones (em inglês) |
| [ROADMAP.md](ROADMAP.md) | Contribuidores | Status, variantes planejadas (em inglês) |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribuidores | Como fazer fork, testar, submeter PRs (em inglês) |
| [masks/README.md](masks/README.md) | Artistas | Como criar `bg_dark.png` do zero (em inglês) |
| [masks/labels/README.md](masks/labels/README.md) | Avançado | Como criar sprite packs (em inglês) |
| [docs/AUDIT.md](docs/AUDIT.md) | Mantenedores | Histórico de auditoria interna (em PT-BR) |

O overlay em runtime só precisa de `index.html` + `masks/bg_dark.png`. Todo o resto é documentação.

---

## Estrutura do projeto

```
.
├── index.html              # Overlay (HTML + CSS + JS inline, arquivo único)
├── masks/
│   ├── bg_dark.png         # Arte de fundo (3840×2160)
│   ├── README.md           # Como criar fundos personalizados
│   └── labels/
│       ├── sprite_example.svg  # Modelo de sprite pack SVG (Fase 5)
│       └── README.md           # Como criar sprite packs
├── CUSTOMIZATION_GUIDE.md  # Cores, formas, labels
├── DESIGN_GUIDE.md         # Grid, proporções, timing
├── LABEL_PACKS_GUIDE.md    # Sprite sheets SVG
├── ROADMAP.md              # Roadmap público
├── CONTRIBUTING.md         # Como contribuir
├── README.md               # Versão em inglês (primária)
├── README.pt.md            # Este arquivo (PT-BR)
├── LICENSE                 # The Unlicense (domínio público)
├── .gitignore
└── docs/
    ├── AUDIT.md            # Histórico de auditoria (PT-BR)
    └── AUDIT_PLAN.md       # Notas de planejamento (PT-BR)
```

---

## Roadmap (público, alto nível)

A base Xbox é o **open core** do projeto. Variantes para outros controles são planejadas para o futuro. Planos detalhados (layouts, mapeamentos, decisões arquiteturais) são mantidos privados pelo mantenedor durante o desenvolvimento.

Variantes em consideração:
- Controle arcade (fightstick) para jogos de luta
- PlayStation DualSense com símbolos ✕ ○ □ △ e touchpad
- Nintendo Pro Controller com layout A/B/X/Y trocado

Veja [ROADMAP.md](ROADMAP.md) para o roadmap público em alto nível.

---

## Contribuindo

Projeto de domínio público. Não há processo formal de contribuição — faça fork, modifique, abra um PR se quiser. Veja [CONTRIBUTING.md](CONTRIBUTING.md) para convenções de código e dicas de teste no OBS.

### Não contribua com

- WebHID (removido — veja [docs/AUDIT.md](docs/AUDIT.md))
- Build pipelines (a restrição single-file HTML é intencional)
- Dependências externas (o overlay deve funcionar offline)

---

## Agradecimentos

- **Etimologia**: *Bubo* — latim para coruja-orelhuda. Na mitologia romana, Bubo era a coruja familiar de Minerva, deusa da sabedoria. A reputação da coruja por visão noturna e vigilância combina com um overlay que observa o seu controle.
- **Etimologia (PT-BR)**: O apelido do mantenedor "Murucutu" vem do Tupi-Guarani para coruja (*Asio clamator* — coruja-orelhuda), ligando as tradições Latina e Tupi.
- **Standard Gamepad API** — por ser estável o suficiente pra dispensar WebHID.
- **OBS Studio** — por ser software excelente e gratuito.

---

## Licença

**[The Unlicense](LICENSE)** — dedicação ao domínio público. Você pode usar, copiar, modificar, publicar, distribuir, sublicenciar e vender este software sem qualquer restrição. Sem necessidade de atribuição.

> O mantenedor oferece **serviços pagos de customização** (overlays temáticos para streamers). A base open-source continua grátis para sempre. Se quiser uma variante customizada (temática, com branding, ou layout proprietário), contate o mantenedor via GitHub ou pelos seus canais habituais de streaming.
