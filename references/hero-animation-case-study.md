# Gallery Ripple + Multi-Focus · Scene Choreography Philosophy

> **A reusable visual-choreography structure** distilled from the ai-design hero animation v9 (25 seconds, 8 scenes).
> This is not an animation production pipeline — it's about **when this kind of choreography is "right" for the scene**.
> Reference: [demos/hero-animation-v9.mp4](../demos/hero-animation-v9.mp4) · [https://www.huasheng.ai/huashu-design-hero/](https://www.huasheng.ai/huashu-design-hero/)

## One-line summary

> **When you have 20+ visually homogeneous assets and the scene needs to "express scale and depth," reach for Gallery Ripple + Multi-Focus before piling on layouts.**

Generic SaaS feature animations, product launches, skill promotion videos, series portfolio showcases — as long as you have enough assets in a consistent style, this structure almost always lands.

---

## What is this technique actually expressing

It's not "showing off assets" — it tells a narrative through **two rhythmic shifts**:

**Beat 1 · Ripple expansion (~1.5s)**: 48 cards spread out from the center to the edges, the audience is hit by the "volume" — "oh, there's *that* much output here."

**Beat 2 · Multi-Focus (~8s, 4 cycles)**: while the camera slowly pans, dim + desaturate the background 4 times and zoom one specific card to the center of the screen — the audience switches from "the impact of quantity" to "the gaze of quality," steady at 1.7s per beat.

**Core narrative structure**: **Scale (Ripple) → Gaze (Focus × 4) → Fade (Walloff)**. These three beats together express "Breadth × Depth" — not just "we can do a lot," but "every single one is worth pausing on."

For comparison, here are some counter-examples:

| Approach | Audience perception |
|------|---------|
| 48 cards in a static grid (no Ripple) | Pretty but no narrative — like a grid screenshot |
| Quick cuts one at a time (no Gallery context) | Like a slideshow, loses the "sense of scale" |
| Just Ripple, no Focus | Impressive but doesn't make any single card memorable |
| **Ripple + Focus × 4 (this recipe)** | **First struck by quantity, then gazing at quality, finally a calm fade — a complete emotional arc** |

---

## Preconditions (all must hold)

This choreography **is not universal**; the 4 conditions below are all required:

1. **Asset count ≥ 20, ideally 30+**
   Fewer than 20 makes the Ripple feel "empty" — density only emerges when every cell of a 48-grid is moving. v9 used 48 cells × 32 images (filled with looping).

2. **Visually consistent style across assets**
   All 16:9 slide previews / all app screenshots / all cover designs — aspect ratio, palette, layout must look like they belong to "one set." Mixing styles makes the Gallery look like a clipboard.

3. **Each asset still readable when enlarged**
   Focus zooms a card to 960px wide; if the original blurs or carries thin information when enlarged, the Focus beat is wasted. A reverse check: can you pick 4 "most representative" assets out of 48? If you can't, the asset quality isn't uniform.

4. **The scene itself is landscape or square, not portrait**
   The Gallery's 3D tilt (`rotateX(14deg) rotateY(-10deg)`) needs horizontal extension; portrait makes the tilt look narrow and awkward.

**Fallback paths when conditions are missing**:

| Missing what | Degrade to what |
|-------|-----------|
| Assets < 20 | Switch to "3-5 assets in a static row + focus each one in turn" |
| Inconsistent styles | Switch to "cover + 3 large chapter images" keynote-style |
| Thin information | Switch to "data-driven dashboard" or "punchline + large type" |
| Portrait scene | Switch to "vertical scroll + sticky cards" |

---

## Technical recipe (v9 production parameters)

### 4-Layer structure

```
viewport (1920×1080, perspective: 2400px)
  └─ canvas (4320×2520, oversized overflow) → 3D tilt + pan
      └─ 8×6 grid = 48 cards (gap 40px, padding 60px)
          └─ img (16:9, border-radius 9px)
      └─ focus-overlay (absolute center, z-index 40)
          └─ img (matches selected slide)
```

**Key**: the canvas is 2.25× the viewport, which is what gives the pan its "peeking into a larger world" feeling.

### Ripple expansion (distance-based delay algorithm)

```js
// Each card's entry time = distance from center × 0.8s delay
const col = i % 8, row = Math.floor(i / 8);
const dc = col - 3.5, dr = row - 2.5;       // offset from center
const dist = Math.hypot(dc, dr);
const maxDist = Math.hypot(3.5, 2.5);
const delay = (dist / maxDist) * 0.8;       // 0 → 0.8s
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
const opacity = expoOut(Math.min(1, localT));
```

**Core parameters**:
- Total duration 1.7s (`T.s3_ripple: [8.3, 10.0]`)
- Max delay 0.8s (center first, corners last)
- 0.7s entry duration per card
- Easing: `expoOut` (burst feel, not smooth)

**Done at the same time**: canvas scale from 1.25 → 0.94 (zoom out to reveal) — a synchronized push-back paired with the appearance.

### Multi-Focus (4-cycle rhythm)

```js
T.focuses = [
  { start: 11.0, end: 12.7, idx: 2  },  // 1.7s
  { start: 13.3, end: 15.0, idx: 3  },  // 1.7s
  { start: 15.6, end: 17.3, idx: 10 },  // 1.7s
  { start: 17.9, end: 19.6, idx: 16 },  // 1.7s
];
```

**Rhythm rule**: 1.7s per focus, 0.6s breathing room between them. Total 8s (11.0–19.6s).

**Inside each focus**:
- In ramp: 0.4s (`expoOut`)
- Hold: middle 0.9s (`focusIntensity = 1`)
- Out ramp: 0.4s (`easeOut`)

**Background changes (this is the key)**:

```js
if (focusIntensity > 0) {
  const dimOp = entryOp * (1 - 0.6 * focusIntensity);  // dim to 40%
  const brt = 1 - 0.32 * focusIntensity;                // brightness 68%
  const sat = 1 - 0.35 * focusIntensity;                // saturate 65%
  card.style.filter = `brightness(${brt}) saturate(${sat})`;
}
```

**Not just opacity — desaturate + darken at the same time**. This makes the foreground overlay's color "pop out" rather than just "get a bit brighter."

**Focus overlay size animation**:
- 400×225 (entry) → 960×540 (hold state)
- Outer ring with 3 layers of shadow + 3px accent-color outline ring, giving a "framed" feeling

### Pan (continuous motion keeps stillness from getting boring)

```js
const panT = Math.max(0, t - 8.6);
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;
```

- Sine wave + linear drift — two layers of motion, not a pure loop, so position is different at every moment
- X/Y at different frequencies (0.12 vs 0.09) avoid a visually-detectable "regular loop"
- Clamped to ±900/500px to prevent drifting out

**Why not pure linear pan**: with pure linear pan the audience can "predict" where it's going; sine + drift makes every second new, and under 3D tilt produces a slight "pleasant motion-sickness" that holds attention.

---

## 5 reusable patterns (distilled from v6→v9 iteration)

### 1. **expoOut as the main easing, not cubicOut**

`easeOut = 1 - (1-t)³` (smooth) vs `expoOut = 1 - 2^(-10t)` (burst then quickly settles).

**Reason for the choice**: expoOut hits 90% in the first 30% of the duration, behaving more like physical damping and matching the intuition of "heavy thing landing." Especially good for:
- Card entry (sense of weight)
- Ripple expansion (shockwave)
- Brand reveal (sense of settling into place)

**When to still use cubicOut**: focus out ramp, symmetric micro-animations.

### 2. **Paper-toned background + terracotta-orange accent (Anthropic lineage)**

```css
--bg: #F7F4EE;        /* warm paper */
--ink: #1D1D1F;       /* near black */
--accent: #D97757;    /* terracotta orange */
--hairline: #E4DED2;  /* warm hairline */
```

**Why**: the warm background still feels "alive" after GIF compression, unlike pure white which feels "screen-y." Terracotta orange as the sole accent runs through terminal prompt, dir-card selection, cursor, brand hyphen, focus ring — every visual anchor strung together by a single color.

**v5 lesson**: added a noise overlay to mimic "paper grain," and GIF frame compression destroyed it (every frame was different). v6 switched to "background color + warm shadow only," kept 90% of the paper feel, GIF size shrank 60%.

### 3. **Two-tier shadow simulating depth, instead of true 3D**

```css
.gallery-card.depth-near { box-shadow: 0 32px 80px -22px rgba(60,40,20,0.22), ... }
.gallery-card.depth-far  { box-shadow: 0 14px 40px -16px rgba(60,40,20,0.10), ... }
```

Use `sin(i × 1.7) + cos(i × 0.73)` as a deterministic algorithm to assign each card to one of three shadow tiers (near/mid/far) — **visually you get a "3D stacked" feel, but the per-frame transform never changes, GPU cost is 0**.

**The cost of true 3D**: each card needs its own `translateZ`, GPU computes 48 transforms + shadow blur every frame. Tried in v4, Playwright struggled even at 25fps. v6's two-tier shadow loses <5% of the perceptible effect at 1/10 the cost.

### 4. **Font-weight changes (font-variation-settings) feel more cinematic than size changes**

```js
const wght = 100 + (700 - 100) * morphP;  // 100 → 700 over 0.9s
wordmark.style.fontVariationSettings = `"wght" ${wght.toFixed(0)}`;
```

The brand wordmark goes from Thin → Bold over 0.9s, with letter-spacing tuned in (-0.045 → -0.048em).

**Why this beats scale up/down**:
- Scale up/down is something the audience has seen too many times — expectations are baked in
- Weight change is "internal fullness," like a balloon being inflated, rather than "being pushed closer"
- Variable fonts only became mainstream in 2020+, so the audience subconsciously feels "modern"

**Limitation**: requires a font that supports variable weights (Inter / Roboto Flex / Recursive, etc.). Static fonts can only fake it (switching between fixed weights causes jumps).

### 5. **Corner Brand low-intensity persistent signature**

The Gallery phase has a small `HUASHU · DESIGN` mark in the upper-left at 16% opacity, 12px font, wide letter-spacing.

**Why include this**:
- After the Ripple burst, the audience can lose focus and forget what they're looking at; a light upper-left mark acts as an anchor
- More refined than a full-screen logo — branding people know that brand signatures don't need to shout
- When the GIF is screenshotted and shared, attribution still survives

**Rule**: only present in the middle section (when the frame is busy); off during the open (don't cover the terminal); off at the end (the brand reveal is the lead).

---

## Counter-examples: when not to use this choreography

**❌ Product demos (where you need to show features)**: Gallery makes each card flash by, the audience doesn't remember any one feature. Use "single-screen focus + tooltip annotations" instead.

**❌ Data-driven content**: the audience needs to read numbers, but the Gallery's quick rhythm doesn't give time. Use "data charts + item-by-item reveal" instead.

**❌ Story narrative**: the Gallery is a "parallel" structure, but stories require "causality." Use keynote chapter transitions instead.

**❌ Only 3-5 assets**: Ripple density is insufficient and looks like a "patch." Use "static layout + highlight one at a time" instead.

**❌ Portrait (9:16)**: the 3D tilt requires horizontal extension; in portrait it feels "tilted wrong" rather than "unfolding."

---

## How to judge whether your task fits this choreography

A three-step quick check:

**Step 1 · Asset count**: count how many similar visual assets you have. < 15 → stop; 15-25 → marginal; 25+ → just go.

**Step 2 · Consistency test**: place 4 random assets side by side — do they look like "one set"? If not → unify the style first or change approach.

**Step 3 · Narrative match**: are you trying to express "Breadth × Depth" (quantity × quality)? Or is it "process," "feature," or "story"? If it's not the first, don't force this template.

If it's yes on all three steps, fork the v6 HTML directly, edit the `SLIDE_FILES` array and the timeline, and you can reuse it. Adjust the palette via `--bg / --accent / --ink` — re-skin without changing the bones.

---

## Related references

- Full technical pipeline: [references/animations.md](animations.md) · [references/animation-best-practices.md](animation-best-practices.md)
- Animation export pipeline: [references/video-export.md](video-export.md)
- Audio configuration (BGM + SFX dual-track): [references/audio-design-rules.md](audio-design-rules.md)
- Apple gallery-style horizontal reference: [references/apple-gallery-showcase.md](apple-gallery-showcase.md)
- Source HTML (v6 + audio integration version): `www.huasheng.ai/huashu-design-hero/index.html`
