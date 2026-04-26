# Cinematic Patterns · Best Practices for Workflow Demos

> Five key patterns for upgrading from "PowerPoint animation" to "keynote-grade cinematic."
> Distilled from the two cinematic demos in the 2026-04 "Let's talk skill" deck (Nuwa workflow + Darwin workflow), reproducible in practice.

---

## 0 · What this document solves

When you need to build a "demo animation that explains a workflow" (typical scenarios: skill workflows, product onboarding, API call sequences, agent task execution), there are two common approaches:

| Paradigm | What it looks like | Result |
|---|---|---|
| **PowerPoint animation** (bad) | step 1 fades in → step 2 fades in → step 3 fades in, with 4 boxes laid out on the same screen | Viewers feel "this is just a PowerPoint with fade effects" — no wow moment |
| **Cinematic** (good) | scene-based, focusing on one thing at a time, with dissolves / focus pulls / morphs between scenes | Viewers feel "this is a keynote presentation clip" and want to screenshot and share |

The root cause of the difference is **not animation technique** but **narrative paradigm**. This document explains how to upgrade from the former to the latter.

---

## 1 · Five core patterns

### Pattern A · Dashboard + Cinematic Overlay two-layer structure

**Problem**: A pure cinematic defaults to a black screen with a single ▶ button. If a user lands on this page and doesn't click, they see nothing.

**Solution**:
```
DEFAULT state (always visible): full static workflow dashboard
  └── viewers immediately understand how this skill / workflow runs

POINT ▶ trigger (overlay floats up): 22-second cinematic
  └── auto-fades back to DEFAULT when finished

```

**Implementation notes**:
- `.dash` is visible by default; `.cinema` defaults to `opacity: 0; pointer-events: none`
- `.play-cta` is a small gold button in the bottom-right corner (not a large central overlay)
- Click → `cinema.classList.add('show')` + `dash.classList.add('hide')`
- Use `requestAnimationFrame` for a single run (not a loop); when finished, `endCinematic()` reverses the state

**Anti-pattern**: default = giant central ▶ overlay covering everything, leaving the page blank until clicked.

---

### Pattern B · Scene-based, NOT step-based

**Problem**: Splitting the animation into "step 1 appears → step 2 appears → ..." is PowerPoint thinking.

**Solution**: Split into 5 scenes, where each scene is an **independent shot**, full-screen, focused on one thing only:

| Scene type | Responsibility | Duration |
|---|---|---|
| 1 · Invoke | User input triggers it (terminal typewriter) | 3-4s |
| 2 · Process | Visualization of the core workflow (unique visual language) | 5-6s |
| 3 · Result/Insight | The key artifact distilled from the process (visualized) | 4-5s |
| 4 · Output | Actual artifact display (file / diff / number) | 3-4s |
| 5 · Hero Reveal | Closing hero moment (large type + value proposition) | 4-5s |

**Total duration ≈ 22 seconds** — this is the tested golden length:
- Shorter than 18 seconds: the PM hasn't gotten oriented before it ends
- Longer than 25 seconds: they lose patience
- 22 seconds is just enough to "hook → unfold → wrap up → leave an impression"

**Implementation notes**:
- `T = { DURATION: 22.0, s1_in: [0, 0.7], s2_in: [3.8, 4.6], ... }` global timeline
- A single `requestAnimationFrame(render)` runs all opacity / transform calculations across scenes
- Don't use chained setTimeouts (they break easily and are hard to debug)
- Easing must use `expoOut` / `easeOut` / cubic-bezier — **linear is forbidden**

---

### Pattern C · Each demo's visual language must be independent

**Problem**: After finishing the first cinematic, you take a shortcut on the second one and reuse the same template (same orbit + pentagon + typewriter + hero type), only swapping the copy.

**Result**: Viewers notice the two skills "look identical," which is equivalent to telling them "these two skills are no different."

**Solution**: Each workflow has a different core metaphor, and the visual language must differ accordingly.

**Comparison case**:

| Dimension | Nuwa (distilling people) | Darwin (optimizing skills) |
|---|---|---|
| Core metaphor | Collect → distill → write | Loop → evaluate → ratchet |
| Visual motion | Float / radiate / pentagon | Loop / rise / contrast |
| Scene 2 | 3D Orbit · 8 archive cards floating in a perspective ellipse | Spin Loop · token runs 5 laps along a 6-node ring |
| Scene 3 | Pentagon · 5 tokens radiate from the center | v1 vs v5 · side-by-side diff (red version vs gold version) |
| Scene 4 | SKILL.md typewriter | Hill-Climb · full-screen curve drawing |
| Scene 5 hero | "21 minutes" serif italic large type | Spinning gear ⚙ + "KEPT +1.1" gold tag |

**Test**: cover the copy and look at only the visuals — can you tell which demo this is? If you can't, you took a shortcut.

---

### Pattern D · Use AI-generated real assets, not emoji or hand-drawn SVG

**Problem**: 3D orbit / gallery scenes need asset fragments to float around. Emoji (📚🎤) are ugly and brand-less; hand-drawn SVG book spines never look like real books.

**Solution**: Use `huashu-gpt-image` to generate a 4×2 grid mega-image (8 thematically related objects · white background · 60px breathing space · unified style), then use `extract_grid.py --mode bbox` to cut out 8 standalone transparent PNGs.

**Prompt notes** (for detailed prompt patterns, see the `huashu-gpt-image` skill):
- IP anchoring ("1960s Caltech archive aesthetic" / "Hearthstone-style consistent treatment")
- White background (easier for cutting; gray backgrounds have nicer atmosphere but make transparent extraction difficult)
- 4×2 not 5×5 (avoids the last-row compression bug)
- Persona finishing ("You are a Wired magazine curator preparing an exhibition photo")

**Anti-pattern**: using emoji as icons, using CSS silhouettes in place of product imagery.

---

### Pattern E · Dual-track BGM + SFX

**Problem**: Animation without sound makes the viewer subconsciously feel "this thing looks like a cheap demo."

**Solution**: a long BGM track + 11 SFX cues.

**Generic SFX cue recipe** (suitable for workflow demos):

| Time | SFX | Trigger scene |
|---|---|---|
| 0.10s | whoosh | terminal rises from below |
| 3.0s | enter | typewriter completes, enter pressed |
| 4.0s | slide-in | scene 2 elements enter |
| 5-9s × 5 times | sparkle | key process moments (each generation / each token / each data point) |
| 14s | click | switch to output scene |
| 17.8s | logo-reveal | hero reveal moment |
| typewriter | type | fires every 2 characters (don't make density too high) |

**Frequency separation**: BGM volume 0.32 (low-frequency floor noise), SFX volume 0.55 (mid-high frequency punch), sparkle 0.7 (must stand out), logo-reveal 0.85 (the strongest hero moment).

**User control**:
- Must have a ▶ start overlay (browser autoplay restrictions)
- Small mute button in the top-right corner (user can toggle silence at any time)
- Don't make it "blast sound the moment they land on this page"

---

## 2 · Static dashboard design notes

The dashboard is Layer 1 of the two-layer structure — even if the PM never clicks ▶, they can still understand this skill.

**Layout**: 3-column grid (or 1 large + 2 small), where each panel solves one problem:

| Panel type | Problem it solves | Example |
|---|---|---|
| **Pipeline / Flow Diagram** | "What is this skill's workflow?" | Nuwa 4-stage pipeline · Darwin autoresearch loop |
| **Snapshot / State** | "What does the actual data look like when it runs?" | Darwin 8-dimension rubric snapshot |
| **Trajectory / Evolution** | "How does it change across multiple runs?" | Darwin 5-generation hill-climb curve |
| **Examples / Gallery** | "What artifacts has it produced already?" | Nuwa 21 personas gallery |
| **Strip · Example I/O** | "What goes in → what comes out" | Nuwa example strip: `› nuwa distill Feynman → feynman.skill (21 min)` |

**Key constraints**:
- Information density must be sufficient (each panel must carry differentiated information)
- But don't pad with data slop (every number must mean something)
- Color palette must match the cinematic (same color family so the transition isn't jarring)

---

## 3 · Debugging and dev tools

Any long animation must come with three dev tools, otherwise debugging will explode.

### Tool 1 · `?seek=N` freezes to second N

```js
const seek = parseFloat(params.get('seek'));
if (!isNaN(seek)) {
  started = true; muted = true;
  frozenT = seek;  // render() uses this t instead of elapsed
  cinema.classList.add('show'); dash.classList.add('hide');
}

// Inside render():
let t = frozenT !== null ? frozenT : (elapsed % T.DURATION);
```

Usage: `http://.../slide.html?seek=12` jumps directly to the frame at second 12 without waiting for playback.

### Tool 2 · `?autoplay=1` skips the ▶ overlay

Convenient for Playwright automated screenshot tests, and for force-starting when embedded in an iframe.

### Tool 3 · Manual REPLAY button

A small button in the top-right corner that lets users / debuggers replay any number of times. CSS:

```css
.replay{position:absolute;top:18px;right:18px;background:rgba(212,165,116,0.1);
  border:1px solid rgba(212,165,116,0.3);color:#D4A574;
  font-family:monospace;font-size:10px;letter-spacing:.28em;text-transform:uppercase;
  padding:6px 12px;border-radius:1px;cursor:pointer;backdrop-filter:blur(6px);z-index:6}
```

---

## 4 · iframe embedding pitfalls (if the cinematic is embedded in a deck)

### Pitfall 1 · The parent window's click zone intercepts buttons inside the iframe

If the deck's index.html adds a "left/right 22vw transparent click zone for paging," it will **cover the ▶ play button inside the iframe** — the user's click on the button gets swallowed as "next page."

**Fix**: add `top: 12vh; bottom: 25vh` to the click zone, leaving 25% of the top and bottom unintercepted, so the central ▶ and bottom-right ▶ inside the iframe can both be clicked.

### Pitfall 2 · iframe steals focus, parent loses keyboard events

Once the user clicks inside the iframe, focus is in the iframe and the parent window can't receive ←/→ keyboard events.

**Fix**:
```js
iframe.addEventListener('load', () => {
  // Inject a keyboard forwarder
  const doc = iframe.contentDocument;
  doc.addEventListener('keydown', (e) => {
    window.dispatchEvent(new KeyboardEvent('keydown', { key: e.key, ... }));
  });
  // After click, pull focus back to the parent window
  doc.addEventListener('click', () => setTimeout(() => window.focus(), 0));
});
```

### Pitfall 3 · file:// vs https:// behavior differences

A cinematic that works fine locally under file:// may break after deployment because:
- Under file://, the iframe contentDocument is same-origin
- Under https:// it's also same-origin (if same host), but audio autoplay restrictions are stricter

**Fix**:
- Before deploying, run a local HTTP test with `python3 -m http.server`
- BGM must wait for the user to click ▶ before calling `bgm.play()`; don't play immediately on page-load

---

## 5 · Anti-pattern quick reference

| ❌ Anti-pattern | ✅ Correct pattern |
|---|---|
| Default = black screen with ▶ overlay | Default = static dashboard, ▶ is supplementary |
| 4 steps laid out horizontally on the same screen with fade-in | 5 scenes with full-screen transitions, each focused on one thing |
| Reuse the template, swap copy, call it a different demo | Each demo has independent visual language (cover the copy and you can still tell them apart) |
| Hand-drawn emoji / SVG as assets | gpt-image-2 mega-image + extract_grid for cutting |
| No BGM, no SFX | Dual-track: BGM + 11 SFX cues |
| Schedule with chained setTimeouts | requestAnimationFrame + global timeline T object |
| Linear animation | Expo / cubic-bezier easing |
| No dev tools | `?seek=N` + `?autoplay=1` + REPLAY button |
| Buttons inside iframe swallowed by parent's click zone | Click zone gets top/bottom margin to make room for buttons |

---

## 6 · Time budget

Following these patterns, a complete cinematic demo (with dashboard) takes:

| Task | Time |
|---|---|
| Design 5-scene narrative + visual language | 30 minutes (be deliberate — this determines independence) |
| Dashboard static layout + content | 1 hour |
| Implement 5 cinematic scenes | 1.5 hours |
| Tune audio cue timing + replay button | 30 minutes |
| Playwright screenshot verification at 5 key moments | 15 minutes |
| **Total per demo** | **3-4 hours** |

The second demo reuses the framework but **its visual language must be independent** — about 2-3 hours.
