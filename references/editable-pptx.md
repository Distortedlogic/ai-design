# Editable PPTX Export: HTML Hard Constraints + Sizing Decisions + Common Errors

This document covers the **`scripts/html2pptx.js` + `pptxgenjs` path**, which translates HTML element-by-element into truly editable PowerPoint text frames. It is also the only path supported by `export_deck_pptx.mjs`.

> **Core premise**: To go down this road, the HTML must follow the 4 constraints below from line one. **Don't write it first and convert later** — retrofitting triggers 2-3 hours of rework (verified the hard way on the 2026-04-20 Options Private Board project).
>
> When visual freedom takes priority (animation / web components / CSS gradients / complex SVG), switch to the PDF path (`export_deck_pdf.mjs` / `export_deck_stage_pdf.mjs`). **Don't** expect pptx export to deliver both visual fidelity and editability — that's a physical constraint of the PPTX file format itself (see "Why the 4 constraints aren't bugs but physical constraints" at the end).

---

## Canvas size: use 960×540pt (LAYOUT_WIDE)

PPTX units are **inches** (physical dimensions), not px. The decision principle: the body's computedStyle dimensions must **match the inch dimensions of the presentation layout** (±0.1", enforced by `validateDimensions` in `html2pptx.js`).

### Comparison of 3 candidate sizes

| HTML body | Physical size | Matching PPT layout | When to choose |
|---|---|---|---|
| **`960pt × 540pt`** | **13.333″ × 7.5″** | **pptxgenjs `LAYOUT_WIDE`** | ✅ **Default recommendation** (modern PowerPoint 16:9 standard) |
| `720pt × 405pt` | 10″ × 5.625″ | Custom | Only when the user specifies the "legacy PowerPoint Widescreen" template |
| `1920px × 1080px` | 20″ × 11.25″ | Custom | ❌ Non-standard size, fonts look unusually small when projected |

**Don't think of HTML size as resolution.** PPTX is a vector document; body size determines **physical dimensions**, not sharpness. An oversized body (20″×11.25″) will not make text crisper — it just makes the pt font sizes relatively smaller against the canvas, looking worse when projected or printed.

### Three equivalent body declarations

```css
body { width: 960pt;  height: 540pt; }    /* clearest, recommended */
body { width: 1280px; height: 720px; }    /* equivalent, px-style */
body { width: 13.333in; height: 7.5in; }  /* equivalent, inch-style */
```

Matching pptxgenjs code:

```js
const pptx = new pptxgen();
pptx.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, no customization needed
```

---

## The 4 hard constraints (violations cause direct errors)

`html2pptx.js` translates HTML DOM into PowerPoint objects element-by-element. PowerPoint's format constraints projected onto HTML = the 4 rules below.

### Rule 1: DIVs cannot contain text directly — must be wrapped in `<p>` or `<h1>`-`<h6>`

```html
<!-- ❌ Wrong: text directly inside div -->
<div class="title">Q3 revenue grew 23%</div>

<!-- ✅ Correct: text inside <p> or <h1>-<h6> -->
<div class="title"><h1>Q3 revenue grew 23%</h1></div>
<div class="body"><p>New users were the main driver</p></div>
```

**Why**: PowerPoint text must live inside a text frame, and text frames correspond to paragraph-level HTML elements (p/h*/li). A bare `<div>` has no corresponding text container in PPTX.

**You can't use `<span>` to carry main text either** — span is an inline element and can't be aligned independently as a text box. Span can only be **nested inside p/h\*** for local styling (bold, color changes).

### Rule 2: CSS gradients are not supported — solid colors only

```css
/* ❌ Wrong */
background: linear-gradient(to right, #FF6B6B, #4ECDC4);

/* ✅ Correct: solid color */
background: #FF6B6B;

/* ✅ If multi-color stripes are required, use flex children with their own solid colors */
.stripe-bar { display: flex; }
.stripe-bar div { flex: 1; }
.red   { background: #FF6B6B; }
.teal  { background: #4ECDC4; }
```

**Why**: PowerPoint shape fill only supports solid/gradient-fill, but pptxgenjs's `fill: { color: ... }` only maps to solid. Going through PowerPoint's native gradient requires a different structure that the toolchain doesn't currently support.

### Rule 3: Background/border/shadow must go on DIVs, not on text tags

```html
<!-- ❌ Wrong: <p> has background color -->
<p style="background: #FFD700; border-radius: 4px;">Key content</p>

<!-- ✅ Correct: outer div carries background/border, <p> is just for text -->
<div style="background: #FFD700; border-radius: 4px; padding: 8pt 12pt;">
  <p>Key content</p>
</div>
```

**Why**: In PowerPoint, shape (rectangles/rounded rectangles) and text frame are two separate objects. HTML's `<p>` only translates to a text frame, while background/border/shadow belong to the shape — they must be written on the **div wrapping the text**.

### Rule 4: DIVs can't use `background-image` — use `<img>` tags

```html
<!-- ❌ Wrong -->
<div style="background-image: url('chart.png')"></div>

<!-- ✅ Correct -->
<img src="chart.png" style="position: absolute; left: 50%; top: 20%; width: 300pt; height: 200pt;" />
```

**Why**: `html2pptx.js` only extracts image paths from `<img>` elements; it does not parse CSS `background-image` URLs.

---

## Path A HTML template skeleton

Each slide is its own HTML file, scope-isolated from the others (avoiding the CSS pollution of single-file decks).

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 960pt; height: 540pt;           /* ⚠️ matches LAYOUT_WIDE */
    font-family: system-ui, -apple-system, "PingFang SC", sans-serif;
    background: #FEFEF9;                    /* solid color, no gradients */
    overflow: hidden;
  }
  /* DIV handles layout/background/border */
  .card {
    position: absolute;
    background: #1A4A8A;                    /* background on the DIV */
    border-radius: 4pt;
    padding: 12pt 16pt;
  }
  /* Text tags only handle font styling, no background/border */
  .card h2 { font-size: 24pt; color: #FFFFFF; font-weight: 700; }
  .card p  { font-size: 14pt; color: rgba(255,255,255,0.85); }
</style>
</head>
<body>

  <!-- Title area: outer div for positioning, inner text tags -->
  <div style="position: absolute; top: 40pt; left: 60pt; right: 60pt;">
    <h1 style="font-size: 36pt; color: #1A1A1A; font-weight: 700;">Use an assertion as the title, not a topic phrase</h1>
    <p style="font-size: 16pt; color: #555555; margin-top: 10pt;">Subtitle for additional context</p>
  </div>

  <!-- Content card: div handles background, h2/p handle text -->
  <div class="card" style="top: 130pt; left: 60pt; width: 240pt; height: 160pt;">
    <h2>Point 1</h2>
    <p>Brief explanatory text</p>
  </div>

  <!-- List: use ul/li, not manual • characters -->
  <div style="position: absolute; top: 320pt; left: 60pt; width: 540pt;">
    <ul style="font-size: 16pt; color: #1A1A1A; padding-left: 24pt; list-style: disc;">
      <li>First point</li>
      <li>Second point</li>
      <li>Third point</li>
    </ul>
  </div>

  <!-- Illustration: use <img> tag, not background-image -->
  <img src="illustration.png" style="position: absolute; right: 60pt; top: 110pt; width: 320pt; height: 240pt;" />

</body>
</html>
```

---

## Common error reference

| Error message | Cause | Fix |
|---------|------|---------|
| `DIV element contains unwrapped text "XXX"` | Bare text inside a div | Wrap text in `<p>` or `<h1>`-`<h6>` |
| `CSS gradients are not supported` | Used linear/radial-gradient | Switch to solid color, or use flex children for segments |
| `Text element <p> has background` | `<p>` tag has a background color | Wrap with `<div>` to carry the background; `<p>` carries only text |
| `Background images on DIV elements are not supported` | div uses background-image | Switch to an `<img>` tag |
| `HTML content overflows body by Xpt vertically` | Content exceeds 540pt | Reduce content or shrink font size, or use `overflow: hidden` to truncate |
| `HTML dimensions don't match presentation layout` | body size doesn't match pres layout | Use `960pt × 540pt` body with `LAYOUT_WIDE`; or defineLayout with custom dimensions |
| `Text box "XXX" ends too close to bottom edge` | Large-font `<p>` is < 0.5 inch from the body bottom | Move it up, leave enough bottom margin; PPT's bottom area itself gets partly obscured |

---

## Basic workflow (3 steps to PPTX)

### Step 1: Write each page as a standalone HTML file following the constraints

```
MyDeck/
├── slides/
│   ├── 01-cover.html    # Each file is a complete 960×540pt HTML
│   ├── 02-agenda.html
│   └── ...
└── illustration/        # All images referenced by <img>
    ├── chart1.png
    └── ...
```

### Step 2: Write a build.js that calls `html2pptx.js`

```js
const pptxgen = require('pptxgenjs');
const html2pptx = require('../scripts/html2pptx.js');  // this skill's script

(async () => {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, matches HTML's 960×540pt

  const slides = ['01-cover.html', '02-agenda.html', '03-content.html'];
  for (const file of slides) {
    await html2pptx(`./slides/${file}`, pres);
  }

  await pres.writeFile({ fileName: 'deck.pptx' });
})();
```

### Step 3: Open and verify

- Open the exported PPTX in PowerPoint/Keynote
- Double-click any text — it should be directly editable (if it's an image, you've violated rule 1)
- Verify overflow: every page should fit within the body and not be cut off

---

## This path vs. other options (when to choose what)

| Need | Choice |
|------|------|
| Colleagues will edit text in the PPTX / sending to non-technical people for further edits | **This path** (editable, requires HTML written from scratch following the 4 constraints) |
| Speech-only / archive, no further edits | `export_deck_pdf.mjs` (multi-file) or `export_deck_stage_pdf.mjs` (single-file deck-stage) — produces a vector PDF |
| Visual freedom takes priority (animation, web components, CSS gradients, complex SVG), accepting non-editable | **PDF** (same as above) — PDF preserves fidelity, is cross-platform, and beats "image PPTX" |

**Never force `html2pptx` on visually-driven HTML** — measured pass rate for visually-driven HTML is < 30%, and reworking the remainder page-by-page is slower than rewriting. For these scenarios, output a PDF, don't squeeze into PPTX.

---

## Fallback: visual mockup already exists but the user insists on editable PPTX

You'll occasionally hit this: you/the user already wrote a visually-driven HTML (with gradients, web components, complex SVG, the works), the right output is PDF, but the user explicitly says "no, it has to be editable PPTX."

**Don't force-run `html2pptx` and hope it passes** — measured pass rate for visually-driven HTML through html2pptx is <30%, and the remaining 70% will throw errors or render incorrectly. The correct fallback is:

### Step 1 · Disclose the limitations first (transparent communication)

In one sentence, tell the user three things:

> "Your current HTML uses [list specifically: gradients / web components / complex SVG / ...]. Converting directly to editable PPTX will fail. I have two options:
> - A. **Output PDF** (recommended) — visuals 100% preserved, recipient can view and print but cannot edit text
> - B. **Use the visual mockup as a blueprint and rewrite an editable HTML version** (preserving the design decisions for color/layout/copy, but reorganizing the HTML structure to comply with the 4 hard constraints, **sacrificing** gradients, web components, complex SVG, and other visual capabilities) → then export to editable PPTX
>
> Which do you prefer?"

Don't soften option B — explicitly state **what will be lost**. Let the user make the tradeoff.

### Step 2 · If the user picks B: AI rewrites it, don't make the user do it

The doctrine here is: **the user provides design intent, you translate it into a compliant implementation**. Don't make the user learn the 4 hard constraints and rewrite it themselves.

Principles when rewriting:
- **Preserve**: color system (primary/secondary/neutral), information hierarchy (title/subtitle/body/annotation), core copy, layout skeleton (top-middle-bottom / left-right / grid), page rhythm
- **Downgrade**: CSS gradient → solid color or flex segments, web component → paragraph-level HTML, complex SVG → simplified `<img>` or solid-color geometry, shadow → remove or weaken to minimum, custom font → align with system fonts
- **Rewrite**: bare text → wrap in `<p>` / `<h*>`, `background-image` → `<img>` tag, background/border on `<p>` → carry on outer div

### Step 3 · Produce a comparison checklist (transparent delivery)

After the rewrite, give the user a before/after comparison so they know which visual details were simplified:

```
Original design → editable version adjustment
- Title area purple gradient → primary color #5B3DE8 solid background
- Data card shadow → removed (replaced with 2pt stroke for differentiation)
- Complex SVG line chart → simplified to <img> PNG (generated from HTML screenshot)
- Hero web component animation → static first frame (web components cannot be translated)
```

### Step 4 · Export & deliver in dual format

- `editable` HTML → run `scripts/export_deck_pptx.mjs` to produce editable PPTX
- **Recommended**: also retain the original visual mockup → run `scripts/export_deck_pdf.mjs` to produce high-fidelity PDF
- Deliver both formats to the user: high-fidelity PDF of the visual mockup + editable PPTX, each serving its purpose

### When to refuse option B outright

In a few scenarios the rewriting cost is too high and you should advise the user to give up on editable PPTX:

- The HTML's core value is animation or interaction (after rewriting only the static first frame remains, info loss 50%+)
- Page count > 30, rewriting cost exceeds 2 hours
- The visual design depends deeply on precise SVG / custom filters (the rewrite would be virtually unrelated to the original)

In these cases tell the user: "The rewriting cost for this deck is too high. I recommend PDF instead of PPTX. If the recipient really requires pptx format, you'll need to accept significant visual simplification — would you like to switch to PDF?"

---

## Why the 4 constraints aren't bugs but physical constraints

These 4 rules are not the result of `html2pptx.js`'s author being lazy — they are projections of the **PowerPoint file format (OOXML) constraints** onto HTML:

- Text in PPTX must live inside a text frame (`<a:txBody>`), corresponding to paragraph-level HTML elements
- PPTX shapes and text frames are two separate objects; you can't paint a background and write text on the same element
- PPTX shape fill has limited gradient support (only certain preset gradients, not arbitrary-angle CSS gradients)
- PPTX picture objects must reference real image files, not CSS properties

Once you understand this, **don't expect the tool to get smarter** — the HTML must adapt to the PPTX format, not the other way around.
