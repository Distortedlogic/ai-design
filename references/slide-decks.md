# Slide Decks: HTML Slide Production Spec

Building slide decks is a high-frequency design scenario. This doc explains how to do HTML slides well — from architecture choice, through single-slide design, to the full PDF/PPTX export path.

**What this skill covers**:
- **HTML presentation version (the base artifact, always required by default)** → one HTML file per slide + `assets/deck_index.html` aggregator, keyboard navigation in the browser, full-screen presenting
- HTML → PDF export → `scripts/export_deck_pdf.mjs` / `scripts/export_deck_stage_pdf.mjs`
- HTML → editable PPTX export → `references/editable-pptx.md` + `scripts/html2pptx.js` + `scripts/export_deck_pptx.mjs` (requires the HTML to follow 4 hard constraints)

> **⚠️ HTML is the base; PDF/PPTX are derivatives.** Whatever the final deliverable format, you **must** first build the HTML aggregated presentation version (`index.html` + `slides/*.html`) — it's the "source" of the slide-deck artifact. PDF/PPTX are one-line snapshots exported from the HTML.
>
> **Why HTML first**:
> - Best for live presenting (projector / screen share, full-screen with keyboard navigation, no dependency on Keynote/PPT)
> - During development each slide can be opened directly by double-click for verification, no need to re-run an export each time
> - It's the sole upstream of PDF/PPTX export (avoids the death-loop "I exported, now I need to change the HTML and re-export")
> - The deliverable can be "HTML + PDF" or "HTML + PPTX" as a pair; the receiver picks whichever they prefer
>
> 2026-04-22 moxt brochure validated this: after building 13 HTML slides + index.html aggregator, `export_deck_pdf.mjs` produced the PDF in one line with zero changes. The HTML version itself is a deliverable you can present from the browser directly.

---

## 🛑 Confirm Delivery Format Before You Start (the hardest checkpoint)

**This decision comes before "single file vs multi-file."** 2026-04-20 options private-board project validated: **not confirming the delivery format before starting work = 2–3 hours of rework.**

### Decision tree (HTML-first architecture)

All deliveries start from the same HTML aggregation pages (`index.html` + `slides/*.html`). The delivery format only determines **constraints on how to write the HTML** and **which export command to run**:

```
[Always default · required] HTML aggregated presentation version (index.html + slides/*.html)
   │
   ├── Only need browser presenting / local HTML archive  → Done here, max visual freedom in HTML
   │
   ├── Also need PDF (printing / sharing in groups / archive) → run export_deck_pdf.mjs, one-line export
   │                                                              HTML stays free, no visual constraints
   │
   └── Also need editable PPTX (colleagues will edit text)  → write HTML to the 4 hard constraints from line 1
                                                                  run export_deck_pptx.mjs for one-line export
                                                                  give up gradients / web components / complex SVG
```

### Opening script (copy-paste)

> Whether the final delivery is HTML, PDF, or PPTX, I'll first build a browseable, presentable HTML aggregation version (`index.html` with keyboard navigation) — that's the always-default base artifact. On top of that I'll then ask you whether to also export PDF/PPTX snapshots.
>
> Which export format do you need?
> - **HTML only** (presenting/archive) → full visual freedom
> - **Plus PDF** → same as above, plus one export command
> - **Plus editable PPTX** (colleagues will edit text in PPT) → I have to write to 4 hard constraints from line 1 of HTML, which sacrifices some visual capabilities (no gradients, no web components, no complex SVG).

### Why "want PPTX = follow 4 hard constraints from the start"

PPTX editability requires `html2pptx.js` to translate the DOM element-by-element into PowerPoint objects. It needs **4 hard constraints**:

1. body fixed at 960pt × 540pt (matches `LAYOUT_WIDE`, 13.333″ × 7.5″, not 1920×1080px)
2. All text wrapped in `<p>` / `<h1>`–`<h6>` (no text directly in a div, no `<span>` carrying main text)
3. `<p>` / `<h*>` itself can't have background/border/shadow (put that on an outer div)
4. `<div>` can't use `background-image` (use an `<img>` tag)
5. No CSS gradients, no web components, no complex SVG decoration

**This skill's HTML defaults to high visual freedom** — heavy use of spans, nested flex, complex SVG, web components (e.g. `<deck-stage>`), CSS gradients — **almost none of which naturally pass html2pptx's constraints** (in practice, visually-driven HTML put through html2pptx has < 30% pass rate).

### The cost comparison of the two real paths (real 2026-04-20 incident)

| Path | What you do | Outcome | Cost |
|------|------|------|------|
| ❌ **Write HTML freely first, retrofit PPTX later** | Single-file deck-stage + heavy SVG/span decoration | To get editable PPTX you only have two options:<br>A. Hand-write a few hundred lines of pptxgenjs with hardcoded coordinates<br>B. Rewrite all 17 HTML slides into Path A format | 2–3 hours of rework, plus the hand-written version's **maintenance cost is forever** (change one word in HTML, manually re-sync to PPTX) |
| ✅ **Write to Path A constraints from step 1** | One HTML per slide + 4 hard constraints + 960×540pt | One command exports a 100% editable PPTX, while also presentable full-screen in the browser (Path A HTML is just standard browser-playable HTML) | Spend 5 extra minutes per slide thinking "how do I wrap this text in `<p>`," zero rework |

### What about hybrid delivery

User says "I want HTML presenting **and** editable PPTX" — **this isn't hybrid**, the PPTX requirement subsumes the HTML requirement. HTML written to Path A is itself presentable full-screen in the browser (just add a `deck_index.html` aggregator). **No additional cost.**

User says "I want PPTX **and** animations / web components" — **this is a real conflict**. Tell the user: if you want editable PPTX, you have to give up these visual capabilities. Make them choose; don't sneak in a hand-written pptxgenjs solution (it becomes perpetual maintenance debt).

### What if you only learn about the PPTX requirement after the fact (emergency rescue)

Rare cases: HTML is already written and only then do you discover the PPTX requirement. Recommended **fallback flow** (full description in `references/editable-pptx.md` end, "Fallback: existing visual draft but user insists on editable PPTX"):

1. **First choice: deliver as PDF** (visuals 100% preserved, cross-platform, recipient can view and print) — if the recipient's actual need is "presenting/archive," PDF is the best deliverable
2. **Second choice: AI rewrites an editable HTML version using the visual draft as the blueprint** → export editable PPTX — preserves color/layout/copy design decisions, sacrifices gradients, web components, complex SVG, etc.
3. **Not recommended: hand-rebuild with pptxgenjs** — positions, fonts, alignment all manually tuned, high maintenance cost, and any HTML word change requires another manual sync

Always tell the user the choices and let them decide. **Never reflexively start hand-writing pptxgenjs** — that's the absolute last-resort fallback.

---

## 🛑 Before Bulk Production: Build a 2-Slide Showcase to Set the Grammar

**For any deck ≥ 5 slides, you absolutely must not write straight from slide 1 to the last slide.** The correct order, validated by the 2026-04-22 moxt brochure work:

1. Pick the **2 slide types with the largest visual difference** to build as a showcase (e.g. "cover" + "mood/quote slide," or "cover" + "product showcase slide")
2. Screenshot it and have the user confirm the grammar (masthead / typography / color / spacing / structure / Chinese-English bilingual ratio)
3. Once the direction is approved, batch-produce the remaining N-2 slides, each reusing the established grammar
4. After all are done, assemble into the HTML aggregation + PDF/PPTX derivatives

**Why**: Writing 13 slides straight through → user says "wrong direction" = 13 slides of rework. Build a 2-slide showcase first → wrong direction = 2 slides of rework. Once the visual grammar is locked in, the decision space for the remaining N slides shrinks dramatically — only "how to fit content in" remains.

**Showcase slide selection principle**: pick the two slides with the most different visual structure. If those two pass = all the intermediate states will pass.

| Deck type | Recommended showcase slide pair |
|-----------|---------------------|
| B2B brochure / product launch | Cover + content slide (philosophy/emotional) |
| Brand launch | Cover + product feature slide |
| Data report | Big-data slide + analytical conclusion slide |
| Course materials | Section cover + concrete topic slide |

---

## 📐 Publication Grammar Template (moxt-tested, reusable)

Suitable for B2B brochures / product launches / long reports. Reusing this structure on every slide = 13 slides of fully consistent visual style, 0 rework.

### Per-slide skeleton

```
┌─ masthead (top strip + horizontal line) ──┐
│  [logo 22-28px] · A Product Brochure                Issue · Date · URL │
├──────────────────────────────────────────┤
│                                          │
│  ── kicker (green dash + uppercase label) │
│  CHAPTER XX · SECTION NAME                 │
│                                          │
│  H1 (Chinese Noto Serif SC 900)           │
│  Key word(s) on brand primary color       │
│                                          │
│  English subtitle (Lora italic, subtitle) │
│  ─────────── divider ──────────            │
│                                          │
│  [actual content: 60/40 two-col / 2x2 grid / list] │
│                                          │
├──────────────────────────────────────────┤
│ section name                     XX / total │
└──────────────────────────────────────────┘
```

### Style conventions (copy directly)

- **H1**: Chinese Noto Serif SC 900, 80–140px depending on info volume, key word(s) on brand primary color (don't pile colors across the whole headline)
- **English subtitle**: Lora italic 26–46px, brand signature words (e.g. "AI team") bold + primary-color italic
- **Body**: Noto Serif SC 17–21px, line-height 1.75–1.85
- **Accent highlights**: bold + primary-color highlights on key words inline, no more than 3 per slide (more loses the anchoring effect)
- **Background**: warm beige #FAFAFA + extremely light radial-gradient noise (`rgba(33,33,33,0.015)`) for paper feel

### The visual lead must vary

If all 13 slides are "text + one screenshot" it gets too monotonous. **Rotate the type of visual lead per slide**:

| Visual type | Suitable section |
|---------|---------------|
| Cover composition (large type + masthead + pillar) | First slide / section cover |
| Single-character portrait (oversized single momo, etc.) | Introducing one concept/character |
| Multi-character group / avatar cards in a row | Team / user case studies |
| Timeline cards progressing | Show "long-term relationship" or "evolution" |
| Knowledge graph / connecting nodes | Show "collaboration" or "flow" |
| Before/After comparison cards + center arrow | Show "change" or "difference" |
| Product UI screenshot + outlined device frame | Concrete feature showcase |
| Big-quote slide (half-page large type) | Mood / problem / quote slide |
| Real-person avatar + quote card (2×2 or 1×4) | User testimonials / use scenarios |
| Big-type closing slide + URL pill button | CTA / closing |

---

## ⚠️ Common Pitfalls (moxt practice summary)

### 1. Emoji don't render when exporting via Chromium / Playwright

Chromium doesn't ship a color emoji font by default; on `page.pdf()` or `page.screenshot()` emoji show as empty boxes.

**Fix**: use Unicode text symbols (`✦` `✓` `✕` `→` `·` `—`) instead, or just go to plain text ("Email · 23" rather than "📧 23 emails").

### 2. `export_deck_pdf.mjs` errors with `Cannot find package 'playwright'`

Cause: ESM module resolution searches upward for `node_modules` from the script's location. The script lives in `~/.claude/skills/ai-design/scripts/`, which has no dependencies.

**Fix**: copy the script into the deck project directory (e.g. `brochure/build-pdf.mjs`), run `npm install playwright pdf-lib` from the project root, then `node build-pdf.mjs --slides slides --out output/deck.pdf`.

### 3. Google Fonts not done loading before the screenshot → Chinese renders as the system default sans

Wait at least `wait-for-timeout=3500` before Playwright screenshots/PDFs so webfonts download and paint. Or self-host the fonts to `shared/fonts/` to reduce network dependency.

### 4. Information density imbalance: too much on a content slide

The first version of the moxt philosophy slide used 2×2 = 4 paragraphs + 3 tenets at the bottom = 7 content blocks, cramped and repetitive. Switching to 1×3 = 3 paragraphs immediately restored breathing room.

**Fix**: keep each slide to "1 core message + 3–4 supporting points + 1 visual lead"; if it overflows, split to a new slide. **Less is more** — the audience looks at a slide for 10 seconds, giving them 1 memory point sticks better than 4.

---

## 🛑 Architecture First: Single-File or Multi-File?

**This choice is the first step in slide-deck work; getting it wrong means hitting the same pitfalls over and over. Read this section before you do anything.**

### Comparing the two architectures

| Dimension | Single file + `deck_stage.js` | **Multi-file + `deck_index.html` aggregator** |
|------|--------------------------|--------------------------------------|
| Code structure | One HTML, every slide is a `<section>` | Each slide its own HTML, `index.html` aggregates via iframe |
| CSS scope | ❌ Global; one slide's styles can affect every slide | ✅ Naturally isolated; each iframe is its own little world |
| Verification granularity | ❌ Have to JS-goTo to switch to a specific slide | ✅ Each slide file opens by double-click in the browser |
| Parallel development | ❌ Single file; multi-agent edits collide | ✅ Multiple agents can build different slides in parallel, zero merge conflicts |
| Debug difficulty | ❌ One CSS error capsizes the whole deck | ✅ One slide's error only affects itself |
| Embedded interaction | ✅ Cross-slide shared state is easy | 🟡 postMessage required between iframes |
| Print to PDF | ✅ Built in | ✅ Aggregator's beforeprint walks the iframes |
| Keyboard navigation | ✅ Built in | ✅ Aggregator built in |

### Which one to pick (decision tree)

```
│ Q: How many slides do you expect?
├── ≤10 slides, needs in-deck animation or cross-slide interaction, pitch deck → single file
└── ≥10 slides, lecture, course materials, long deck, multi-agent parallel → multi-file (recommended)
```

**Default to multi-file**. It's not a "fallback"; it's **the main path for long decks and team collaboration**. Reason: every advantage of single-file (keyboard navigation, printing, scaling) is also present in multi-file, while multi-file's scope isolation and verifiability can't be retrofitted into single-file.

### Why this rule is so hard (real incident log)

Single-file architecture once stepped on four pitfalls in a row during the AI-psychology lecture deck production:

1. **CSS specificity override**: `.emotion-slide { display: grid }` (specificity 10) overrode `deck-stage > section { display: none }` (specificity 2), causing all slides to render simultaneously stacked.
2. **Shadow DOM slot rules suppressed by outer CSS**: `::slotted(section) { display: none }` couldn't hold against an outer rule's override; sections refused to hide.
3. **localStorage + hash navigation race**: after refresh, instead of jumping to the hash position it stuck at the old position recorded in localStorage.
4. **Verification cost is high**: had to `page.evaluate(d => d.goTo(n))` to screenshot a particular slide, twice as slow as `goto(file://.../slides/05-X.html)`, and frequently errors.

The root cause across all of them is **a single global namespace** — the multi-file architecture eliminates these problems at the physical layer.

---

## Path A (default): Multi-File Architecture

### Directory structure

```
MyDeck/
├── index.html              # Copied from assets/deck_index.html, edit MANIFEST
├── shared/
│   ├── tokens.css          # Shared design tokens (palette / type scale / common chrome)
│   └── fonts.html          # <link> imports for Google Fonts (include on each slide)
└── slides/
    ├── 01-cover.html       # Each file is a complete 1920×1080 HTML
    ├── 02-agenda.html
    ├── 03-problem.html
    └── ...
```

### Per-slide template skeleton

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>P05 · Chapter Title</title>
<link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
<link rel="stylesheet" href="../shared/tokens.css">
<style>
  /* Styles unique to this slide. Any class name used here won't pollute other slides. */
  body { padding: 120px; }
  .my-thing { ... }
</style>
</head>
<body>
  <!-- 1920×1080 content (the body's width/height are locked in tokens.css) -->
  <div class="page-header">...</div>
  <div>...</div>
  <div class="page-footer">...</div>
</body>
</html>
```

**Key constraints**:
- `<body>` is the canvas — lay out directly on it. Don't wrap in `<section>` or any other wrapper.
- `width: 1920px; height: 1080px` is locked by the `body` rule in `shared/tokens.css`.
- Import `shared/tokens.css` for shared design tokens (palette, type scale, page-header/footer, etc.).
- Each slide writes its own font `<link>` (a separate font import is cheap, and it ensures every slide is independently openable).

### Aggregator: `deck_index.html`

**Copy directly from `assets/deck_index.html`**. You only have to change one thing — the `window.DECK_MANIFEST` array, listing all slide filenames in order with human-readable labels:

```js
window.DECK_MANIFEST = [
  { file: "slides/01-cover.html",    label: "Cover" },
  { file: "slides/02-agenda.html",   label: "Agenda" },
  { file: "slides/03-problem.html",  label: "Problem Statement" },
  // ...
];
```

The aggregator has built in: keyboard navigation (←/→/Home/End/number keys/P to print), scale + letterbox, bottom-right counter, localStorage memory, hash navigation, print mode (walks iframes to output the PDF page-by-page).

### Per-slide verification (multi-file's killer advantage)

Every slide is a standalone HTML. **As soon as you finish a slide, double-click it in the browser**:

```bash
open slides/05-personas.html
```

Playwright screenshots are also direct `goto(file://.../slides/05-personas.html)` — no JS jumping, no interference from other slides' CSS. This makes the "edit one, verify one" workflow nearly free.

### Parallel development

Hand each slide as a task to a different agent and run them in parallel — the HTML files are independent, no merge conflicts. Long decks can be built in 1/N the time using this parallel approach.

### What goes in `shared/tokens.css`

Only put things that are **genuinely cross-slide common**:

- CSS variables (palette, type scale, spacing scale)
- `body { width: 1920px; height: 1080px; }` and similar canvas locks
- `.page-header` / `.page-footer` chrome that appears identically on every slide

**Don't** drop single-slide layout classes in here — that regresses to the global pollution problem of single-file architecture.

---

## Path B (small decks): Single File + `deck_stage.js`

For ≤10 slides, when you need cross-slide shared state (e.g. a single React tweaks panel controlling all slides), or for tightly-packed pitch deck demos.

### Basic usage

1. Read the contents of `assets/deck_stage.js`, embed in HTML's `<script>` (or `<script src="deck_stage.js">`)
2. In the body, wrap slides in `<deck-stage>`
3. 🛑 **The script tag must come after `</deck-stage>`** (see hard constraint below)

```html
<body>

  <deck-stage>
    <section>
      <h1>Slide 1</h1>
    </section>
    <section>
      <h1>Slide 2</h1>
    </section>
  </deck-stage>

  <!-- ✅ Correct: script after deck-stage -->
  <script src="deck_stage.js"></script>

</body>
```

### 🛑 Script Position Hard Constraint (real 2026-04-20 incident)

**You cannot put `<script src="deck_stage.js">` in `<head>`.** Even though it can define `customElements` while in `<head>`, the parser triggers `connectedCallback` the moment it parses the `<deck-stage>` start tag — at which point the child `<section>`s aren't parsed yet, `_collectSlides()` returns an empty array, the counter shows `1 / 0`, and all slides render stacked at once.

**Three compliant patterns** (pick any):

```html
<!-- ✅ Most recommended: script after </deck-stage> -->
</deck-stage>
<script src="deck_stage.js"></script>

<!-- ✅ Also fine: script in head with defer -->
<head><script src="deck_stage.js" defer></script></head>

<!-- ✅ Also fine: module scripts are naturally deferred -->
<head><script src="deck_stage.js" type="module"></script></head>
```

`deck_stage.js` itself includes a `DOMContentLoaded` deferred-collection defense — even script-in-head won't fully blow up — but `defer` or putting it at the bottom of body is still cleaner; don't lean on the defense branch.

### ⚠️ Single-file Architecture's CSS Trap (must read)

The most common pitfall in single-file architecture — **`display` getting stolen by per-slide styles**.

Common wrong pattern 1 (writing display: flex directly on section):

```css
/* ❌ External CSS specificity 2 overrides shadow DOM's ::slotted(section){display:none} (also 2) */
deck-stage > section {
  display: flex;            /* All slides will render stacked simultaneously! */
  flex-direction: column;
  padding: 80px;
  ...
}
```

Common wrong pattern 2 (section has a higher-specificity class):

```css
.emotion-slide { display: grid; }   /* Specificity: 10, even worse */
```

Both make **all slides render stacked simultaneously** — the counter may show `1 / 10` pretending things are normal, but visually the first slide overlays the second overlays the third.

### ✅ Starter CSS (copy at start of work, no pitfalls)

**The section itself** only manages "visible/invisible"; **layout (flex/grid, etc.) goes on `.active`**:

```css
/* section only defines non-display common styles */
deck-stage > section {
  background: var(--paper);
  padding: 80px 120px;
  overflow: hidden;
  position: relative;
  /* ⚠️ Don't write display here! */
}

/* Lock down "non-active means hidden" — specificity + weight double belt-and-suspenders */
deck-stage > section:not(.active) {
  display: none !important;
}

/* Active slide gets the desired display + layout */
deck-stage > section.active {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Print mode: all slides should display, override :not(.active) */
@media print {
  deck-stage > section { display: flex !important; }
  deck-stage > section:not(.active) { display: flex !important; }
}
```

Alternative: **put per-slide flex/grid on an inner wrapper `<div>`**, so the section itself is only ever a `display: block/none` switch. This is the cleanest approach:

```html
<deck-stage>
  <section>
    <div class="slide-content flex-layout">...</div>
  </section>
</deck-stage>
```

### Custom dimensions

```html
<deck-stage width="1080" height="1920">
  <!-- 9:16 vertical -->
</deck-stage>
```

---

## Slide Labels

Both `deck_stage` and `deck_index` label each slide (shown by the counter). Give them **more meaningful** labels:

**Multi-file**: in `MANIFEST`, write `{ file, label: "04 Problem Statement" }`
**Single-file**: on the section, add `<section data-screen-label="04 Problem Statement">`

**Key: slide numbering starts at 1, not 0**.

When the user says "slide 5" they mean the 5th one, never array index `[4]`. Humans don't speak 0-indexed.

---

## Speaker Notes

**Off by default**; only add when the user explicitly asks.

With speaker notes you can shrink the on-slide text to a minimum and focus on impactful visuals — the notes carry the full script.

### Format

**Multi-file**: in `index.html`'s `<head>`, write:

```html
<script type="application/json" id="speaker-notes">
[
  "Script for slide 1...",
  "Script for slide 2...",
  "..."
]
</script>
```

**Single-file**: same location.

### Notes writing tips

- **Complete**: not an outline, the actual words you'd say
- **Conversational**: like normal speech, not written prose
- **Aligned**: the Nth array entry corresponds to the Nth slide
- **Length**: 200–400 characters works best
- **Emotional beats**: mark stresses, pauses, emphasis points

---

## Slide Design Patterns

### 1. Establish a system (mandatory)

After exploring the design context, **first state the system you'll use, verbally**:

```markdown
Deck system:
- Background colors: at most 2 (90% white + 10% dark for section dividers)
- Type: display in Instrument Serif, body in Geist Sans
- Rhythm: section dividers full-bleed color + white text; regular slides white-bg
- Imagery: hero slides use full-bleed photo; data slides use a chart

I'll work to this system; tell me if there's an issue.
```

After the user confirms, then proceed.

### 2. Common slide layouts

- **Title slide**: solid bg + huge title + subtitle + author/date
- **Section divider**: color bg + section number + section title
- **Content slide**: white bg + title + 1–3 bullet points
- **Data slide**: title + large chart/number + brief explanation
- **Image slide**: full-bleed photo + small caption at the bottom
- **Quote slide**: whitespace + huge quote + attribution
- **Two-column**: left/right comparison (vs / before-after / problem-solution)

A deck uses at most 4–5 layouts.

### 3. Scale (reiterating)

- Body min **24px**, ideal 28–36px
- Headline **60–120px**
- Hero type **180–240px**
- Slides are read from 10 meters away — type must be big enough

### 4. Visual rhythm

Decks need **intentional variety**:

- Color rhythm: mostly white-bg + occasional color section divider + occasional dark passage
- Density rhythm: a few text-heavy + a few image-heavy + a few quote/whitespace
- Type-size rhythm: normal headlines + occasional giant hero text

**Don't make every slide look the same** — that's a PPT template, not a design.

### 5. Spatial breathing (must-read for data-dense slides)

**The classic beginner mistake**: cramming everything that could fit into a single slide.

Information density ≠ effective communication. Lecture / presentation decks especially need restraint:

- List/matrix slides: don't draw all N elements at the same size. Use **primary/secondary layering** — make the 5 you're focusing on today big as the lead, shrink the remaining 16 as background hints.
- Big-number slides: the number is the visual lead. The surrounding caption shouldn't exceed 3 lines, or the audience's eyes ping-pong.
- Quote slides: leave whitespace between the quote and the attribution; don't stick them together.

Self-check against "is the data the lead?" and "is the text crammed together?" — keep editing until the whitespace makes you a little uncomfortable.

---

## Print to PDF

**Multi-file**: `deck_index.html` already handles the `beforeprint` event, outputting the PDF page-by-page.

**Single-file**: `deck_stage.js` handles it the same way.

The print styles are already written; no need to write extra `@media print` CSS.

---

## Export to PPTX / PDF (self-serve scripts)

HTML-first is the first-class citizen. But users often need PPTX/PDF delivery. Two general scripts are provided that **work with any multi-file deck**, located under `scripts/`:

### `export_deck_pdf.mjs` — Export a vector PDF (multi-file architecture)

```bash
node scripts/export_deck_pdf.mjs --slides <slides-dir> --out deck.pdf
```

**Features**:
- Text is **kept as vectors** (selectable, searchable)
- Visual fidelity 100% (Playwright's embedded Chromium renders, then prints)
- **No need to change a single character of the HTML**
- Each slide its own `page.pdf()`, then merged with `pdf-lib`

**Dependencies**: `npm install playwright pdf-lib`

**Limit**: PDF text isn't editable — to change it, edit the HTML.

### `export_deck_stage_pdf.mjs` — Single-file deck-stage architecture only ⚠️

**When to use**: deck is a single HTML file + `<deck-stage>` web component wrapping N `<section>`s (i.e. Path B architecture). Here `export_deck_pdf.mjs`'s "one `page.pdf()` per HTML" doesn't work; you need this dedicated script.

```bash
node scripts/export_deck_stage_pdf.mjs --html deck.html --out deck.pdf
```

**Why you can't reuse export_deck_pdf.mjs** (real 2026-04-20 incident log):

1. **Shadow DOM beats `!important`**: deck-stage's shadow CSS contains `::slotted(section) { display: none }` (only the active one is `display: block`). Even adding `@media print { deck-stage > section { display: block !important } }` in light DOM can't override it — when `page.pdf()` triggers print media, Chromium's final render only shows the active slide, so the **whole PDF is just 1 page** (the current active slide repeated).

2. **Looping goto still emits 1 page per iteration**: the intuitive fix "navigate to each `#slide-N` then `page.pdf({pageRanges:'1'})`" also fails — because the print CSS rule `deck-stage > section { display: block }` outside shadow DOM gets overridden too, and the final render is always the first item in the section list (not the one you navigated to). 17 loops produce 17 copies of the P01 cover.

3. **Absolute children leak to the next page**: even if you successfully render all sections, if a section's `position` is `static`, its absolute-positioned `cover-footer`/`slide-footer` will position relative to the initial containing block — when print forces section to 1080px height, the absolute footer can be pushed to the next page (resulting in PDF having 1 more page than section count, with that extra page only containing the orphan footer).

**Fix strategy** (already implemented in the script):

```js
// After opening the HTML, use page.evaluate to lift the sections out of the deck-stage slot,
// attach them directly under body in a regular div, and inline-style them to ensure position:relative + fixed dimensions
await page.evaluate(() => {
  const stage = document.querySelector('deck-stage');
  const sections = Array.from(stage.querySelectorAll(':scope > section'));
  document.head.appendChild(Object.assign(document.createElement('style'), {
    textContent: `
      @page { size: 1920px 1080px; margin: 0; }
      html, body { margin: 0 !important; padding: 0 !important; }
      deck-stage { display: none !important; }
    `,
  }));
  const container = document.createElement('div');
  sections.forEach(s => {
    s.style.cssText = 'width:1920px!important;height:1080px!important;display:block!important;position:relative!important;overflow:hidden!important;page-break-after:always!important;break-after:page!important;background:#F7F4EF;margin:0!important;padding:0!important;';
    container.appendChild(s);
  });
  // Disable page-break on the last slide to avoid a trailing blank page
  sections[sections.length - 1].style.pageBreakAfter = 'auto';
  sections[sections.length - 1].style.breakAfter = 'auto';
  document.body.appendChild(container);
});

await page.pdf({ width: '1920px', height: '1080px', printBackground: true, preferCSSPageSize: true });
```

**Why this works**:
- Pulling sections out of the shadow DOM slot into a normal light-DOM div completely bypasses the `::slotted(section) { display: none }` rule
- Inline `position: relative` makes absolute children position relative to the section, no overflow
- `page-break-after: always` makes the browser put each section on its own page when printing
- `:last-child` not paginated avoids the trailing blank page

**When verifying with `mdls -name kMDItemNumberOfPages`**: macOS Spotlight metadata is cached. After re-writing the PDF you must run `mdimport file.pdf` to force a refresh, or it shows the old page count. Use `pdfinfo` or `pdftoppm` to count pages for the real number.

---

### `export_deck_pptx.mjs` — Export editable PPTX

```bash
# Only mode: native editable text frames (fonts fall back to system fonts)
node scripts/export_deck_pptx.mjs --slides <dir> --out deck.pptx
```

How it works: `html2pptx` reads computedStyle element-by-element to translate the DOM into PowerPoint objects (text frame / shape / picture). Text becomes a real text frame, double-click in PPT to edit.

**Hard constraints** (HTML must satisfy these or that slide is skipped; full description in `references/editable-pptx.md`):
- All text must be inside `<p>` / `<h1>`–`<h6>` / `<ul>` / `<ol>` (no bare-text divs)
- `<p>` / `<h*>` itself can't have background/border/shadow (put on outer div)
- Don't use `::before`/`::after` to insert decorative text (pseudo-elements can't be extracted)
- Inline elements (span/em/strong) can't have margin
- No CSS gradients (not renderable)
- div doesn't use `background-image` (use `<img>`)

The script has a built-in **auto-preprocessor** — it auto-wraps "bare text inside leaf divs" into `<p>` (preserving class). This solves the most common violation (bare text). But other violations (border on p, margin on span, etc.) still require source-level HTML compliance.

**Font fallback caveat**:
- Playwright uses webfonts to measure text-box dimensions; PowerPoint/Keynote uses the local machine's fonts to render
- When they differ you get **overflow or misalignment** — eyeball every slide
- Recommendation: install the HTML's fonts on the target machine, or fall back to `system-ui`

**Don't take this path for visual-priority scenarios** → use `export_deck_pdf.mjs` for PDF instead. PDF has 100% visual fidelity, vector, cross-platform, searchable text — it's the true home of visual-priority decks, not some "uneditable compromise."

### Make HTML export-friendly from the start

For maximally robust decks: **write HTML to the editable 4 hard constraints from the start**. Then `export_deck_pptx.mjs` passes everything directly. Extra cost is small:

```html
<!-- ❌ Bad -->
<div class="title">Key Findings</div>

<!-- ✅ Good (wrapped in p, class inherits) -->
<p class="title">Key Findings</p>

<!-- ❌ Bad (border on p) -->
<p class="stat" style="border-left: 3px solid red;">41%</p>

<!-- ✅ Good (border on outer div) -->
<div class="stat-wrap" style="border-left: 3px solid red;">
  <p class="stat">41%</p>
</div>
```

### When to pick which

| Scenario | Recommendation |
|------|------|
| For organizers / archival | **PDF** (universal, high fidelity, searchable text) |
| Sending to collaborators for text edits | **PPTX editable** (accept font fallback) |
| Live presenting, no content edits | **PDF** (vector fidelity, cross-platform) |
| HTML is the primary delivery medium | Play directly in browser; export is just backup |

## The Deep Path to Editable PPTX (long-term projects only)

If your deck will be maintained long-term, edited repeatedly, with team collaboration — recommendation: **write HTML to html2pptx constraints from the start**, so `export_deck_pptx.mjs` passes everything directly. See `references/editable-pptx.md` for details (4 hard constraints + HTML template + common-error quick reference + fallback flow for existing visual drafts).

---

## Common Issues

**Multi-file: an iframe slide won't open / shows white**
→ Check whether `MANIFEST`'s `file` path is correct relative to `index.html`. Use browser DevTools to see if the iframe's src is directly accessible.

**Multi-file: a slide's styles conflict with another slide**
→ Impossible (iframe isolation). If it feels like a conflict, it's caching — Cmd+Shift+R to hard-refresh.

**Single-file: multiple slides render stacked**
→ CSS specificity issue. See the "Single-file Architecture's CSS Trap" section above.

**Single-file: scaling looks wrong**
→ Check whether all slides hang directly under `<deck-stage>` as `<section>`. No intermediate `<div>` wrapper allowed.

**Single-file: jump to a specific slide**
→ Add a hash to the URL: `index.html#slide-5` jumps to slide 5.

**Both architectures: text positions inconsistent across screens**
→ Use fixed dimensions (1920×1080) and `px` units, not `vw`/`vh` or `%`. Scaling is handled uniformly.

---

## Verification Checklist (must pass when deck is done)

1. [ ] Open `index.html` (or main HTML) directly in the browser; check that the first slide has no broken images and the fonts have loaded
2. [ ] Press → through every slide; no blank slides, no layout breaks
3. [ ] Press P for print preview; each slide is exactly one A4 (or 1920×1080) with no clipping
4. [ ] Pick 3 slides at random and Cmd+Shift+R hard-refresh; localStorage memory works correctly
5. [ ] Playwright batch screenshot (multi-file: iterate `slides/*.html`; single-file: use goTo to switch); eyeball-review them all
6. [ ] Search for any `TODO` / `placeholder` remnants and confirm they're cleaned up
