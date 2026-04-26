# In-Depth Design Review Guide

> Detailed reference for Phase 7. Provides scoring rubrics, scenario-specific priorities, and a common-issues checklist.

---

## Scoring Rubric Details

### 1. Philosophy Alignment

| Score | Standard |
|------|------|
| 9-10 | The design fully embodies the chosen philosophy's core spirit; every detail has a philosophical basis |
| 7-8 | The overall direction is correct, core characteristics are in place, with only individual details drifting |
| 5-6 | Intent is visible, but execution mixes in elements from other styles — not pure enough |
| 3-4 | Surface mimicry only; the philosophical core is not understood |
| 1-2 | Essentially unrelated to the chosen philosophy |

**Review checklist**:
- Does it use the signature techniques of that designer / studio?
- Are color, typography, and layout aligned with that philosophical system?
- Are there self-contradicting elements? (e.g., choosing Kenya Hara but stuffing the page full of content)

### 2. Visual Hierarchy

| Score | Standard |
|------|------|
| 9-10 | The user's eye flows naturally along the designer's intended path; information acquisition has zero friction |
| 7-8 | Primary/secondary relationships are clear; 1-2 minor hierarchy ambiguities |
| 5-6 | Title and body are distinguishable, but the middle hierarchy is muddled |
| 3-4 | Information is flat, with no clear visual entry point |
| 1-2 | Chaos — the user doesn't know where to look first |

**Review checklist**:
- Is the size contrast between title and body sufficient? (at least 2.5×)
- Do color / weight / size establish 3-4 clear hierarchy levels?
- Is whitespace guiding the eye?
- The "squint test": squint your eyes — is the hierarchy still clear?

### 3. Craft Quality

| Score | Standard |
|------|------|
| 9-10 | Pixel-precise; alignment, spacing, and color have no flaws |
| 7-8 | Polished overall, with 1-2 small alignment / spacing issues |
| 5-6 | Mostly aligned, but spacing isn't uniform and color usage isn't systematic enough |
| 3-4 | Obvious alignment errors, chaotic spacing, too many colors |
| 1-2 | Rough — looks like a draft |

**Review checklist**:
- Is a uniform spacing system used (e.g., an 8pt grid)?
- Is spacing consistent across elements of the same type?
- Is the number of colors controlled? (typically no more than 3-4)
- Is the type family unified? (typically no more than 2)
- Is edge alignment precise?

### 4. Functionality

| Score | Standard |
|------|------|
| 9-10 | Every design element serves the goal; zero redundancy |
| 7-8 | Function-driven, with a small amount of removable decoration |
| 5-6 | Usable, but obvious decorative elements distract attention |
| 3-4 | Form over function; users have to work to find information |
| 1-2 | Buried by decoration; loses the ability to convey information |

**Review checklist**:
- If you remove any one element, will the design get worse? (If not, remove it.)
- Is the CTA / key information in the most prominent position?
- Are there elements added "because they look nice"?
- Does information density match the medium? (Slides shouldn't be too dense; PDFs can be denser.)

### 5. Originality

| Score | Standard |
|------|------|
| 9-10 | Refreshing — finds a unique expression within the chosen philosophical framework |
| 7-8 | Has its own ideas; not just template assembly |
| 5-6 | By-the-book; looks like a template |
| 3-4 | Heavy use of clichés (e.g., gradient orbs to represent AI) |
| 1-2 | Pure template or asset assembly |

**Review checklist**:
- Does it avoid common clichés? (See "Common Issues Checklist" below.)
- Within the design philosophy, is there personal expression?
- Are there "unexpected but well-justified" design decisions?

---

## Scenario-Specific Review Priorities

Different output types call for different review priorities:

| Scenario | Most important | Secondary | Can be relaxed |
|------|-----------|--------|--------|
| Public-account cover / inline image | Originality, visual hierarchy | Philosophy alignment | Functionality (single image, no interaction) |
| Infographic | Functionality, visual hierarchy | Craft quality | Originality (accuracy first) |
| Slides / Keynote | Visual hierarchy, functionality | Craft quality | Originality (clarity first) |
| PDF / whitepaper | Craft quality, functionality | Visual hierarchy | Originality (professionalism first) |
| Landing page / marketing site | Functionality, visual hierarchy | Originality | — (everything matters) |
| App UI | Functionality, craft quality | Visual hierarchy | Philosophy alignment (usability first) |
| Xiaohongshu inline image | Originality, visual hierarchy | Philosophy alignment | Craft quality (mood first) |

---

## Top 10 Common Design Problems

### 1. AI-tech clichés
**Problem**: Gradient orbs, digital rain, blue circuit boards, robot faces.
**Why it's a problem**: Users are visually fatigued by these and can't distinguish you from anyone else.
**Fix**: Replace literal symbols with abstract metaphors (e.g., a metaphor for "conversation" rather than a chat-bubble icon).

### 2. Insufficient size hierarchy
**Problem**: Too small a gap between title and body (<2.5×).
**Why it's a problem**: Users can't quickly locate key information.
**Fix**: Title should be at least 3× the body (e.g., body 16px → title 48-64px).

### 3. Too many colors
**Problem**: 5+ colors used with no primary/secondary structure.
**Why it's a problem**: Visual chaos, weak brand presence.
**Fix**: Limit to 1 primary + 1 secondary + 1 accent + grayscale.

### 4. Inconsistent spacing
**Problem**: Element spacing is arbitrary, no system.
**Why it's a problem**: Looks unprofessional; visual rhythm is broken.
**Fix**: Establish an 8pt grid system (use only 8/16/24/32/48/64px).

### 5. Insufficient whitespace
**Problem**: Every space is filled with content.
**Why it's a problem**: Information feels crowded, causing reading fatigue and ironically lowering communication efficiency.
**Fix**: Whitespace should occupy at least 40% of the area (60%+ for minimalist styles).

### 6. Too many fonts
**Problem**: 3+ fonts in use.
**Why it's a problem**: Visual noise, weakened cohesion.
**Fix**: At most 2 fonts (1 for titles + 1 for body); use weight and size for variation.

### 7. Inconsistent alignment
**Problem**: Some elements left-aligned, some centered, some right-aligned.
**Why it's a problem**: Destroys the sense of visual order.
**Fix**: Pick one alignment (left-aligned recommended) and apply it globally.

### 8. Decoration overpowering content
**Problem**: Background patterns / gradients / shadows steal the spotlight from the main content.
**Why it's a problem**: Inverted priorities — the user came for information, not decoration.
**Fix**: Ask "if I delete this decoration, does the design get worse?" If not, delete it.

### 9. Cyber-neon overuse
**Problem**: Deep blue background (#0D1117) + neon glow effects.
**Why it's a problem**: A default aesthetic forbidden zone (this skill's taste baseline), and it's already become one of the biggest clichés — users may override based on their own brand.
**Fix**: Pick a more recognizable palette (refer to the 20-style color systems).

### 10. Information density mismatched to medium
**Problem**: A whole page of text on a slide / 10 elements crammed into a cover image.
**Why it's a problem**: Different media have different optimal densities.
**Fix**:
- Slides: one core point per slide
- Cover image: one visual focal point
- Infographic: layered presentation
- PDF: can be denser, but needs clear navigation

---

## Review Output Template

```
## Design Review Report

**Overall Score**: X.X/10 [Excellent (8+) / Good (6-7.9) / Needs work (4-5.9) / Failing (<4)]

**Subscores**:
- Philosophy alignment: X/10 [one-line note]
- Visual hierarchy: X/10 [one-line note]
- Craft quality: X/10 [one-line note]
- Functionality: X/10 [one-line note]
- Originality: X/10 [one-line note]

### Strengths (Keep)
- [Concretely point out what works, in design language]

### Issues (Fix)
[Sorted by severity]

**1. [Issue name]** — ⚠️ Critical / ⚡ Important / 💡 Polish
- Current: [describe the current state]
- Problem: [why this is a problem]
- Fix: [specific action with values]

### Quick Wins
If you only have 5 minutes, prioritize these 3 things:
- [ ] [Highest-impact fix]
- [ ] [Second most important]
- [ ] [Third most important]
```

---

**Version**: v1.0
**Last updated**: 2026-02-13
