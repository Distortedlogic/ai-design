# Design Philosophy Showcases — Sample Asset Index

> 8 scenarios x 3 styles = 24 prefabricated design samples
> Used during Phase 3 to recommend a design direction by directly showing "this is what this style looks like in practice."

## Style Notes

| Code | School | Style Name | Visual Character |
|------|--------|------------|------------------|
| **Pentagram** | Information Architecture | Pentagram / Michael Bierut | Black-and-white restraint, Swiss grid, strong typographic hierarchy, #E63946 red accent |
| **Build** | Minimalism | Build Studio | Luxury-grade whitespace (70%+), subtle weights (200-600), #D4A574 warm gold, refined |
| **Takram** | Eastern Philosophy | Takram | Soft tech feel, natural colors (beige/grey/green), rounded corners, charts as art |

## Scenario Quick Reference

### Content Design Scenarios

| # | Scenario | Spec | Pentagram | Build | Takram |
|---|----------|------|-----------|-------|--------|
| 1 | WeChat public account cover | 1200x510 | `cover/cover-pentagram` | `cover/cover-build` | `cover/cover-takram` |
| 2 | PPT data page | 1920x1080 | `ppt/ppt-pentagram` | `ppt/ppt-build` | `ppt/ppt-takram` |
| 3 | Vertical infographic | 1080x1920 | `infographic/infographic-pentagram` | `infographic/infographic-build` | `infographic/infographic-takram` |

### Website Design Scenarios

| # | Scenario | Spec | Pentagram | Build | Takram |
|---|----------|------|-----------|-------|--------|
| 4 | Personal homepage | 1440x900 | `website-homepage/homepage-pentagram` | `website-homepage/homepage-build` | `website-homepage/homepage-takram` |
| 5 | AI navigation site | 1440x900 | `website-ai-nav/ainav-pentagram` | `website-ai-nav/ainav-build` | `website-ai-nav/ainav-takram` |
| 6 | AI writing tool | 1440x900 | `website-ai-writing/aiwriting-pentagram` | `website-ai-writing/aiwriting-build` | `website-ai-writing/aiwriting-takram` |
| 7 | SaaS landing page | 1440x900 | `website-saas/saas-pentagram` | `website-saas/saas-build` | `website-saas/saas-takram` |
| 8 | Developer documentation | 1440x900 | `website-devdocs/devdocs-pentagram` | `website-devdocs/devdocs-build` | `website-devdocs/devdocs-takram` |

> Each entry has both an `.html` (source) and a `.png` (screenshot) file.

## Usage Notes

### Referencing during Phase 3 recommendations
After recommending a design direction, you can show the prefabricated screenshot for the corresponding scenario:
```
"This is what the Pentagram style looks like for a WeChat public account cover -> [show cover/cover-pentagram.png]"
"This is what a PPT data page feels like in Takram style -> [show ppt/ppt-takram.png]"
```

### Scenario matching priority
1. The user's requested scenario has an exact match -> show that scenario directly.
2. No exact match but the type is similar -> show the closest scenario (e.g., "product website" -> show the SaaS landing page).
3. No match at all -> skip the prefabricated samples and go straight to Phase 3.5 live generation.

### Side-by-side comparison
The 3 styles for the same scenario work well shown side by side, giving the user an intuitive comparison:
- "This is the same WeChat public account cover, executed in 3 different styles."
- Display order: Pentagram (rational restraint) -> Build (luxurious minimalism) -> Takram (soft warmth).

## Content Details

### WeChat public account cover (cover/)
- Subject: Claude Code Agent workflow — 8 parallel Agent architecture
- Pentagram: huge red "8" + Swiss grid lines + data bars
- Build: ultra-thin "Agent" weight floating in 70% whitespace + warm gold hairlines
- Takram: 8-node radial flowchart as art piece + beige background

### PPT data page (ppt/)
- Subject: GLM-4.7 open-source model coding capability breakthrough (AIME 95.7 / SWE-bench 73.8% / tau-squared-Bench 87.4)
- Pentagram: 260px "95.7" anchor + red/grey/light-grey contrast bar chart
- Build: three sets of 120px ultra-thin numerals floating + warm gold gradient comparison bars
- Takram: SVG radar chart + tri-color overlay + rounded data cards

### Vertical infographic (infographic/)
- Subject: AI memory system CLAUDE.md optimized from 93KB to 22KB
- Pentagram: huge "93->22" numerals + numbered blocks + CSS data bars
- Build: extreme whitespace + soft-shadow cards + warm gold connecting lines
- Takram: SVG ring chart + organic curved flowchart + frosted-glass cards

### Personal homepage (website-homepage/)
- Subject: indie developer Alex Chen's portfolio homepage
- Pentagram: 112px large name + Swiss grid columns + editorial numerals
- Build: glassmorphism nav + floating stats cards + ultra-thin weights
- Takram: paper texture + small round avatar + hairline dividers + asymmetric layout

### AI navigation site (website-ai-nav/)
- Subject: AI Compass — directory of 500+ AI tools
- Pentagram: square-cornered search box + numbered tool list + uppercase category tags
- Build: rounded search box + refined white tool cards + pill tags
- Takram: organic offset card layout + soft category tags + chart-style connections

### AI writing tool (website-ai-writing/)
- Subject: Inkwell — AI writing assistant
- Pentagram: 86px headline + wireframe editor mockup + grid-based feature columns
- Build: floating editor card + warm gold CTA + luxurious writing experience
- Takram: poetic serif headline + organic editor + flowchart

### SaaS landing page (website-saas/)
- Subject: Meridian — business intelligence analytics platform
- Pentagram: black-and-white columns + structured dashboard + 140px "3x" anchor
- Build: floating dashboard cards + SVG area chart + warm gold gradient
- Takram: rounded bar charts + flow nodes + soft earth tones

### Developer documentation (website-devdocs/)
- Subject: Nexus API — unified AI model gateway
- Pentagram: left sidebar nav + square-cornered code blocks + red string highlighting
- Build: centered floating code cards + soft shadows + warm gold icons
- Takram: beige code blocks + flowchart connections + dashed-border feature cards

## File Statistics

- HTML source files: 24
- PNG screenshots: 24
- Total assets: 48 files

---

**Version**: v1.0
**Created**: 2026-02-13
**Used for**: ai-design skill, Phase 3 recommendation step
