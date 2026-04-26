# Apple Gallery Showcase · Gallery Display Wall Animation Style

> Inspiration: Claude Design landing-page hero video + Apple product page "work wall" displays
> Production source: ai-design release hero v5
> Use cases: **product launch hero animations, skill capability demos, portfolio displays** — any scenario where you need to display "many high-quality outputs" simultaneously and direct viewer attention

---

## Trigger judgment: when to use this style

**Suitable**:
- 10+ real outputs to display on the same screen (PPT, App, web page, infographic)
- Audience is professional (developers, designers, product managers), sensitive to "craft"
- The vibe you want to convey is "restrained, exhibition-style, premium, with spatial depth"
- Need focus and overview to coexist (look at details without losing the whole)

**Not suitable**:
- Single-product focus (use the frontend-design product hero template)
- Emotional / story-heavy animations (use the timeline narrative template)
- Small screens / vertical orientation (the tilted perspective gets blurry on small screens)

---

## Core visual tokens

```css
:root {
  /* Light gallery palette */
  --bg:         #F5F5F7;   /* main canvas background — Apple website gray */
  --bg-warm:    #FAF9F5;   /* warm off-white variant */
  --ink:        #1D1D1F;   /* primary text color */
  --ink-80:     #3A3A3D;
  --ink-60:     #545458;
  --muted:      #86868B;   /* secondary text */
  --dim:        #C7C7CC;
  --hairline:   #E5E5EA;   /* card 1px border */
  --accent:     #D97757;   /* terracotta orange — Claude brand */
  --accent-deep:#B85D3D;

  --serif-cn: "Noto Serif SC", "Songti SC", Georgia, serif;
  --serif-en: "Source Serif 4", "Tiempos Headline", Georgia, serif;
  --sans:     "Inter", -apple-system, "PingFang SC", system-ui;
  --mono:     "JetBrains Mono", "SF Mono", ui-monospace;
}
```

**Key principles**:
1. **Never use a pure black background**. Black makes the work look like a movie, not "a work output that could be adopted"
2. **Terracotta orange is the only accent hue**, everything else is grayscale + white
3. **Three-font stack** (English serif + Chinese serif + sans + mono) creates a "publication" rather than "internet product" vibe

---

## Core layout patterns

### 1. Floating cards (the basic unit of the entire style)

```css
.gallery-card {
  background: #FFFFFF;
  border-radius: 14px;
  padding: 6px;                          /* padding is the "matting paper" */
  border: 1px solid var(--hairline);
  box-shadow:
    0 20px 60px -20px rgba(29, 29, 31, 0.12),   /* primary shadow, soft and long */
    0 6px 18px -6px rgba(29, 29, 31, 0.06);     /* secondary near-light, creating float */
  aspect-ratio: 16 / 9;                  /* unified slide ratio */
  overflow: hidden;
}
.gallery-card img {
  width: 100%; height: 100%;
  object-fit: cover;
  border-radius: 9px;                    /* slightly smaller radius than the card, visually nested */
}
```

**Counter-example**: don't tile flush (no padding, no border, no shadow) — that's infographic-density expression, not exhibition.

### 2. 3D-tilted work wall

```css
.gallery-viewport {
  position: absolute; inset: 0;
  overflow: hidden;
  perspective: 2400px;                   /* deeper perspective, tilt isn't exaggerated */
  perspective-origin: 50% 45%;
}
.gallery-canvas {
  width: 4320px;                         /* canvas = 2.25× viewport */
  height: 2520px;                        /* leave pan space */
  transform-origin: center center;
  transform: perspective(2400px)
             rotateX(14deg)              /* tilt back */
             rotateY(-10deg)             /* turn left */
             rotateZ(-2deg);             /* slight tilt to remove over-regularity */
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 60px;
}
```

**Parameter sweet spot**:
- rotateX: 10-15deg (more and it looks like a VIP banquet backdrop)
- rotateY: ±8-12deg (left-right symmetry feel)
- rotateZ: ±2-3deg (the "this isn't machine-placed" human touch)
- perspective: 2000-2800px (under 2000 fish-eyes, over 3000 approaches orthographic)

### 3. 2×2 four-corner convergence (selection scene)

```css
.grid22 {
  display: grid;
  grid-template-columns: repeat(2, 800px);
  gap: 56px 64px;
  align-items: start;
}
```

Each card slides in toward the center from its corresponding corner (tl/tr/bl/br) + fades in. The corresponding `cornerEntry` vectors:

```js
const cornerEntry = {
  tl: { dx: -700, dy: -500 },
  tr: { dx:  700, dy: -500 },
  bl: { dx: -700, dy:  500 },
  br: { dx:  700, dy:  500 },
};
```

---

## Five core animation patterns

### Pattern A · Four-corner convergence (0.8-1.2s)

4 elements slide in from the four corners of the viewport, simultaneously scaling 0.85→1.0, with ease-out. Suitable for an opening that "shows multi-directional choices".

```js
const inP = easeOut(clampLerp(t, start, end));
card.style.transform = `translate3d(${(1-inP)*ce.dx}px, ${(1-inP)*ce.dy}px, 0) scale(${0.85 + 0.15*inP})`;
card.style.opacity = inP;
```

### Pattern B · Selected zoom + others slide out (0.8s)

The selected card scales 1.0→1.28; other cards fade out + blur + drift back to the corners:

```js
// Selected
card.style.transform = `translate3d(${cellDx*outP}px, ${cellDy*outP}px, 0) scale(${1 + 0.28*easeOut(zoomP)})`;
// Unselected
card.style.opacity = 1 - outP;
card.style.filter = `blur(${outP * 1.5}px)`;
```

**Key**: unselected ones must blur, not pure fade. Blur simulates depth-of-field, visually "pushing out" the selected one.

### Pattern C · Ripple expansion (1.7s)

From center outward, delay by distance; each card fades in in sequence + scales from 1.25x down to 0.94x ("camera pulls back"):

```js
const col = i % COLS, row = Math.floor(i / COLS);
const dc = col - (COLS-1)/2, dr = row - (ROWS-1)/2;
const dist = Math.sqrt(dc*dc + dr*dr);
const delay = (dist / maxDist) * 0.8;
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
card.style.opacity = easeOut(Math.min(1, localT));

// At the same time the whole gallery scales 1.25→0.94
const galleryScale = 1.25 - 0.31 * easeOut(rippleProgress);
```

### Pattern D · Sinusoidal Pan (continuous drift)

Use sine wave + linear drift in combination, avoiding the "has-a-start-and-end" loop feel of a marquee:

```js
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;    // horizontal drift left
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;    // vertical drift up
const clampedX = Math.max(-900, Math.min(900, panX));   // prevent edge exposure
```

**Parameters**:
- Sine period `0.09-0.15 rad/s` (slow, about 30-50 seconds per swing)
- Linear drift `5-8 px/s` (slower than a viewer's blink)
- Amplitude `120-220 px` (large enough to feel, small enough not to make dizzy)

### Pattern E · Focus Overlay (focus switching)

**Key design**: focus overlay is a **flat element** (not tilted), floating above the tilted canvas. The selected slide scales from tile position (about 400×225) to screen center (960×540); the background canvas doesn't tilt-change but **dims to 45%**:

```js
// Focus overlay (flat, centered)
focusOverlay.style.width = (startW + (endW - startW) * focusIntensity) + 'px';
focusOverlay.style.height = (startH + (endH - startH) * focusIntensity) + 'px';
focusOverlay.style.opacity = focusIntensity;

// Background cards dim, but remain visible (key! don't make it 100% mask)
card.style.opacity = entryOp * (1 - 0.55 * focusIntensity);   // 1 → 0.45
card.style.filter = `brightness(${1 - 0.3 * focusIntensity})`;
```

**Sharpness iron rules**:
- The focus overlay's `<img>` must `src` the original image directly — **don't reuse the compressed thumbnail in the gallery**
- Preload all original images into a `new Image()[]` array in advance
- The overlay's own `width/height` is computed per frame; the browser resamples the original each frame

---

## Timeline architecture (reusable skeleton)

```js
const T = {
  DURATION: 25.0,
  s1_in: [0.0, 0.8],    s1_type: [1.0, 3.2],  s1_out: [3.5, 4.0],
  s2_in: [3.9, 5.1],    s2_hold: [5.1, 7.0],  s2_out: [7.0, 7.8],
  s3_hold: [7.8, 8.3],  s3_ripple: [8.3, 10.0],
  panStart: 8.6,
  focuses: [
    { start: 11.0, end: 12.7, idx: 2  },
    { start: 13.3, end: 15.0, idx: 3  },
    { start: 15.6, end: 17.3, idx: 10 },
    { start: 17.9, end: 19.6, idx: 16 },
  ],
  s4_walloff: [21.1, 21.8], s4_in: [21.8, 22.7], s4_hold: [23.7, 25.0],
};

// Core easings
const easeOut = t => 1 - Math.pow(1 - t, 3);
const easeInOut = t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2;
function lerp(time, start, end, fromV, toV, easing) {
  if (time <= start) return fromV;
  if (time >= end) return toV;
  let p = (time - start) / (end - start);
  if (easing) p = easing(p);
  return fromV + (toV - fromV) * p;
}

// A single render(t) function reads the timestamp and writes all elements
function render(t) { /* ... */ }
requestAnimationFrame(function tick(now) {
  const t = ((now - startMs) / 1000) % T.DURATION;
  render(t);
  requestAnimationFrame(tick);
});
```

**Architectural essence**: **all state is derived from timestamp t** — no state machine, no setTimeout. This way:
- Jumping to any moment via `window.__setTime(12.3)` works instantly (handy for playwright frame-by-frame capture)
- Looping is naturally seamless (t mod DURATION)
- Any frame can be frozen for debugging

---

## Craft details (easy to overlook but fatal)

### 1. SVG noise texture

Light backgrounds fear "too flat" most. Layer on an extremely faint fractalNoise:

```html
<style>
.stage::before {
  content: '';
  position: absolute; inset: 0;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 0.078  0 0 0 0 0.078  0 0 0 0 0.074  0 0 0 0.035 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  opacity: 0.5;
  pointer-events: none;
  z-index: 30;
}
</style>
```

Looks no different at first glance — remove it and you'll know it was there.

### 2. Corner brand mark

```html
<div class="corner-brand">
  <div class="mark"></div>
  <div>AI-DESIGN</div>
</div>
```

```css
.corner-brand {
  position: absolute; top: 48px; left: 72px;
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--muted);
}
```

Only shown in the work-wall scene, fade in / out. Like a museum exhibit label.

### 3. Brand collapse wordmark

```css
.brand-wordmark {
  font-family: var(--sans);
  font-size: 148px;
  font-weight: 700;
  letter-spacing: -0.045em;   /* negative letter-spacing is key, makes type tight enough to read as a logo */
}
.brand-wordmark .accent {
  color: var(--accent);
  font-weight: 500;           /* accent character is actually thinner — visual contrast */
}
```

`letter-spacing: -0.045em` is the standard practice for large type on Apple product pages.

---

## Common failure modes

| Symptom | Cause | Fix |
|---|---|---|
| Looks like a PPT template | Cards have no shadow / hairline | Add two-layer box-shadow + 1px border |
| Tilt feels cheap | Only used rotateY without rotateZ | Add ±2-3deg rotateZ to break the regularity |
| Pan feels "stuttery" | Used setTimeout or CSS keyframes loop | Use rAF + sin/cos continuous functions |
| Type unclear when focused | Reused gallery tile's low-res image | Independent overlay + direct original src |
| Background too empty | Pure color `#F5F5F7` | Layer on SVG fractalNoise at 0.5 opacity |
| Type feels too "internet" | Only Inter | Add Serif (Chinese + English) + mono triple stack |

---

## References

- Full implementation sample: `/Users/alchain/Documents/Writing/01-Public Account Writing/Project/2026.04-ai-design release/Artwork/hero-animation-v5.html`
- Original inspiration: claude.ai/design hero video
- Reference aesthetic: Apple product pages, Dribbble shot collection pages

When you have a "many high-quality outputs to display" animation requirement, copy the skeleton from this file directly, swap content + adjust timing.
