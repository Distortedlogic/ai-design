# ai-design

English translation of the Claude Code skill at [`alchaincyf/huashu-design`](https://github.com/alchaincyf/huashu-design).

The upstream skill ships its operative `SKILL.md` and reference docs in Chinese. This repo translates them to English while keeping the same architecture, the same starter assets, and the same export toolchain. Binary assets (BGM, SFX, showcase PNGs) and code (`.jsx`, `.js`, `.mjs`, `.sh`, `.py`) are copied verbatim from upstream.

All design and engineering credit goes to the upstream author. License terms inherited from upstream — see [`LICENSE`](LICENSE).

## What it does

3 to 30 minutes — interactive HTML prototypes, motion design (MP4/GIF with BGM), HTML slide decks (convertible to editable PPTX), parameterized design variants, print-grade infographics, and a 5-axis design critique. Triggered by natural-language prompts; outputs land as files in your working directory.

| Capability | Output |
|---|---|
| Interactive prototype | Single-file HTML, real device bezel, clickable, Playwright-verified |
| Slide deck | HTML deck + editable PPTX (real text frames) |
| Motion design | MP4 (25fps base / 60fps interpolated) + GIF + BGM |
| Design variations | Side-by-side panels + live `Tweaks` parameter switching |
| Infographic | Print-quality typography, exports PDF/PNG/SVG |
| Direction advisor | 3 differentiated directions from 20 design philosophies |
| Expert critique | Radar chart + Keep/Fix/Quick Wins punch list |

## Repository layout

```
ai-design/
├── SKILL.md           main spec read by the agent
├── LICENSE            personal-use license, inherited from upstream
├── assets/            starter components, BGM, SFX, showcase samples
├── references/        drill-down docs the SKILL.md points to
├── scripts/           export toolchain (HTML→MP4/GIF/PPTX/PDF)
├── demos/             capability demos
└── test-prompts.json  prompt regression suite
```

## Core principles enforced by SKILL.md

- **Principle #0 — Fact verification first.** Any claim about specific products, versions, or release dates triggers a `WebSearch` before anything else. Memory-only assertions are banned.
- **Core asset protocol.** When a brand is named, a five-step asset gathering process runs: ask → search official channels → download (logo / product shots / UI screenshots) → verify and extract → freeze to a `brand-spec.md`.
- **Junior designer workflow.** Stage delivery as assumptions → placeholders → variations → real content; show early instead of one-shotting.
- **Anti AI-slop.** No purple gradients, no Inter-as-display, no SVG-silhouette stand-ins for real product images.
- **Direction advisor fallback.** Vague briefs trigger 3 differentiated direction proposals from 20 philosophies before any execution.

## Origin

`huashu-design` was built after Anthropic launched Claude Design. The upstream author deconstructed Claude Design's system prompts (brand-asset protocol, component mechanics, design philosophies), structured them into a spec, and shipped it as a Claude Code skill. This repo carries that work to English readers verbatim.
