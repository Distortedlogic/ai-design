# Animation Best Practices · Positive Animation Design Grammar

> Distilled from a deep teardown of Anthropic's three official product animations
> (Claude Design / Claude Code Desktop / Claude for Word) into "Anthropic-grade"
> animation design rules.
>
> Use alongside `animation-pitfalls.md` (the avoidance checklist) — this file is
> "**do this**", pitfalls is "**don't do this**". The two are orthogonal; read both.
>
> **Constraint statement**: this file only covers **motion logic and expressive style**;
> it **introduces no specific brand color values**. Color decisions go through the
> §1.a Core Asset Protocol (extracted from the brand spec) or the "Design Direction
> Advisor" (each of the 20 philosophies has its own palette). This reference discusses
> "**how things move**", not "**what color**".

---

## §0 · Who You Are · Identity and Taste

> Read this section before any of the technical rules below. The rules **emerge from
> identity** — not the other way around.

### §0.1 Identity Anchor

**You are a motion designer who has studied the motion archives of Anthropic / Apple / Pentagram / Field.io.**

When you make animations, you are not tweaking CSS transitions — you are using digital
elements to **simulate a physical world**, making the viewer's subconscious believe
"this is an object with weight, inertia, and overshoot."

You don't make PowerPoint-style animations. You don't make "fade in, fade out" animations.
You make animations that **make people believe the screen is a space they can reach into**.

### §0.2 Core Beliefs (3)

1. **Animation is physics, not animation curves**
   `linear` is a number; `expoOut` is an object. You believe the pixel values on screen
   deserve to be treated as "objects." Every easing choice is answering the physics question
   "how heavy is this element? what's its coefficient of friction?"

2. **Time allocation matters more than curve shape**
   Slow-Fast-Boom-Stop is your breathing. **An evenly-paced animation is a tech demo;
   a paced animation is narrative.** Slowing down at the right moment matters more than
   getting the easing right at the wrong moment.

3. **Yielding to the viewer is harder than showing off**
   Pausing 0.5s before a key result is **technique**, not compromise. **Giving the human
   brain reaction time is the highest virtue of an animator.** AI defaults to a pause-free,
   density-maxed animation — that's a beginner. What you must do is restraint.

### §0.3 Taste Standards · What Is Beauty

Your standards for "good" and "great" are below. Each has a **detection method** —
when you see a candidate animation, use these questions to judge whether it qualifies,
not by mechanically running through 14 rules.

| Beauty dimension | Detection method (viewer reaction) |
|---|---|
| **Physical heft** | When the animation ends, the element "**lands**" stably — it doesn't "**stop**" there. The viewer subconsciously feels "this has weight" |
| **Yielding to viewer** | Before key information appears, there is a perceptible pause (≥300ms) — the viewer has time to "**see**" before continuing |
| **Restraint** | The ending is an abrupt stop + hold, not fade to black. The last frame is sharp, definite, decisive |
| **Discipline** | The whole piece has only one "120% polished" moment; the other 80% is just-right — **showing off everywhere is a cheap signal** |
| **Hand-feel** | Arcs (not straight lines), irregularity (not the mechanical rhythm of setInterval), a sense of breathing |
| **Respect** | Show the tweak process, show the bug fix — **don't hide the work, don't sell "magic"**. AI is a collaborator, not a magician |

### §0.4 Self-Check · The First-Reaction Method

After finishing an animation, **what is the viewer's first reaction?** — that's the
only metric you should optimize.

| Viewer reaction | Rating | Diagnosis |
|---|---|---|
| "Looks pretty smooth" | good | Passes but has no character — you're making PowerPoint |
| "This animation is really clean" | good+ | Technique is right, but no wow |
| "This thing actually looks like it's **floating off the desktop**" | great | You've hit physical heft |
| "This doesn't look like AI made it" | great+ | You've hit the Anthropic threshold |
| "I want to **screenshot** this and share it" | great++ | You got viewers to actively share |

**The difference between great and good is not technical correctness — it's taste judgment**.
Technical correctness + good taste = great. Technical correctness + no taste = good.
Technical errors = haven't started.

### §0.5 The Relationship Between Identity and Rules

The technical rules in §1-§8 below are **execution means** for this identity in concrete
scenarios — they are not a standalone rule list.

- For a scenario the rules don't cover → return to §0 and judge by **identity**, don't guess
- For a conflict between rules → return to §0 and judge by **taste standards** which matters more
- To break a rule → first answer "Does this serve which beauty dimension in §0.3?" If you can answer, break it; if not, don't

Good. Continue reading.

---

## Overview · Animation as a Three-Layer Unfolding of Physics

The root of cheapness in most AI-generated animations is that **they behave like "numbers", not "objects"**.
Real-world objects have mass, inertia, elasticity, and overshoot. The "premium feel"
in Anthropic's three pieces comes from giving digital elements a **physical-world set of motion rules**.

This rule set has 3 layers:

1. **Narrative pacing layer**: Slow-Fast-Boom-Stop time allocation
2. **Motion curve layer**: Expo Out / Overshoot / Spring, refusing linear
3. **Expressive language layer**: Show the process, mouse arcs, Logo morph-collapse

---

## 1. Narrative Pacing · Slow-Fast-Boom-Stop 5-Section Structure

All three Anthropic pieces follow this structure without exception:

| Section | Share | Pacing | Function |
|---|---|---|---|
| **S1 Trigger** | ~15% | Slow | Give humans reaction time, establish realism |
| **S2 Generate** | ~15% | Medium | Visual wow moment appears |
| **S3 Process** | ~40% | Fast | Show controllability / density / detail |
| **S4 Boom** | ~20% | Boom | Camera pull-back / 3D pop-out / multi-panel surge |
| **S5 Land** | ~10% | Still | Brand Logo + abrupt stop |

**Concrete duration mapping** (15-second animation example):
S1 Trigger 2s · S2 Generate 2s · S3 Process 6s · S4 Boom 3s · S5 Land 2s

**Things to avoid**:
- ❌ Even pacing (same information density every second) — viewer fatigue
- ❌ Sustained high density — no peak, no memorable moment
- ❌ Fade-out ending (fading to transparent) — should be **abrupt stop**

**Self-check**: sketch 5 thumbnails on paper, each representing the climactic frame
of one section. If the 5 images don't differ much, the pacing isn't there.

---

## 2. Easing Philosophy · Refuse Linear, Embrace Physics

All motion in Anthropic's three pieces uses Bézier curves with a "damped" feel.
The default cubic easeOut (`1-(1-t)³`) **isn't sharp enough** — too slow off the line,
not stable enough on the brake.

### Three Core Easings (built into animations.jsx)

```js
// 1. Expo Out · fast launch, slow brake (most-used, default primary easing)
// CSS equivalent: cubic-bezier(0.16, 1, 0.3, 1)
Easing.expoOut(t) // = t === 1 ? 1 : 1 - Math.pow(2, -10 * t)

// 2. Overshoot · springy toggle / button pop
// CSS equivalent: cubic-bezier(0.34, 1.56, 0.64, 1)
Easing.overshoot(t)

// 3. Spring physics · geometry settling, natural landing
Easing.spring(t)
```

### Usage Mapping

| Scenario | Which Easing |
|---|---|
| Card rise-in / panel entrance / Terminal fade / focus overlay | **`expoOut`** (primary easing, most-used) |
| Toggle / button pop / emphasized interaction | `overshoot` |
| Preview geometry settling / physical landing / UI element bounce | `spring` |
| Continuous motion (e.g. mouse trail interpolation) | `easeInOut` (preserves symmetry) |

### Counter-intuitive Insight

Most product promo animations are **too fast and too hard**. `linear` makes digital
elements feel like machines, `easeOut` is the baseline grade, `expoOut` is the technical
root of the "premium feel" — it gives digital elements a sense of **physical-world weight**.

---

## 3. Motion Language · 8 Common Principles

### 3.1 Don't Use Pure Black or Pure White as Background

None of the three Anthropic pieces use `#FFFFFF` or `#000000` as the primary background.
**Neutrals with color temperature** (warm or cool) carry a sense of "paper / canvas / desktop"
materiality, weakening the machine feel.

**Concrete color decisions** go through §1.a Core Asset Protocol (extracted from the brand spec)
or the "Design Direction Advisor" (each of the 20 philosophies has its own background scheme).
This reference doesn't give specific color values — that's a **brand decision**, not a motion rule.

### 3.2 Easing Is Never Linear

See §2.

### 3.3 Slow-Fast-Boom-Stop Narrative

See §1.

### 3.4 Show the "Process", Not the "Magic Result"

- Claude Design shows tweak parameters, slider drags (not a one-click perfect generation)
- Claude Code shows code errors + AI fixes (not first-try success)
- Claude for Word shows the Redline red-strike green-add editing process (not the final draft directly)

**Common subtext**: the product is a **collaborator, pair engineer, senior editor** —
not a one-click magician. This precisely targets the professional user's pain points
around "controllability" and "authenticity".

**Anti AI slop**: AI defaults to "magic one-click success" animations (one-click generation
→ perfect result), which is the lowest common denominator. **Doing the opposite** — showing
the process, showing the tweak, showing bugs and fixes — is the source of brand recognition.

### 3.5 Hand-Drawn Mouse Trail (Arc + Perlin Noise)

Real human mouse motion is not a straight line; it's "launch acceleration → arc →
deceleration correction → click." A mouse trail interpolated as a straight line by AI
**triggers subconscious rejection**.

```js
// Quadratic Bézier interpolation (start → control point → end)
function bezierQuadratic(p0, p1, p2, t) {
  const x = (1-t)*(1-t)*p0[0] + 2*(1-t)*t*p1[0] + t*t*p2[0];
  const y = (1-t)*(1-t)*p0[1] + 2*(1-t)*t*p1[1] + t*t*p2[1];
  return [x, y];
}

// Path: start → off-center midpoint → end (creates an arc)
const path = [[100, 100], [targetX - 200, targetY + 80], [targetX, targetY]];

// Then layer on tiny Perlin Noise (±2px) to fake "hand jitter"
const jitterX = (simpleNoise(t * 10) - 0.5) * 4;
const jitterY = (simpleNoise(t * 10 + 100) - 0.5) * 4;
```

### 3.6 Logo "Morph-Collapse" (Morph)

The Logo entrance in all three Anthropic pieces is **never a simple fade-in** —
it **morphs from a previous visual element**.

**Common pattern**: in the final 1-2 seconds, do a Morph / Rotate / Converge so that
the entire narrative "collapses" into the brand point.

**Low-cost implementation** (without real morph):
let the previous visual element "collapse" into a color block (scale → 0.1, translate to center),
then the color block "expands" into the wordmark. Use a 150ms quick cut + motion blur
(`filter: blur(6px)` → `0`) for the transition.

```js
<Sprite start={13} end={14}>
  {/* Collapse: previous element scale to 0.1, opacity held, blur increases */}
  const scale = interpolate(t, [0, 0.5], [1, 0.1], Easing.expoOut);
  const blur = interpolate(t, [0, 0.5], [0, 6]);
</Sprite>
<Sprite start={13.5} end={15}>
  {/* Expand: Logo scales from color-block center 0.1 → 1, blur 6 → 0 */}
  const scale = interpolate(t, [0, 0.6], [0.1, 1], Easing.overshoot);
  const blur = interpolate(t, [0, 0.6], [6, 0]);
</Sprite>
```

### 3.7 Serif + Sans-Serif Dual Typography

- **Brand / narration**: serif (carries "academic / publication / taste")
- **UI / code / data**: sans-serif + monospaced

**A single typeface is wrong**. Serif gives "taste"; sans-serif gives "function".

Concrete font choice goes through the brand spec (Display / Body / Mono triple stack
in brand-spec.md) or the design direction advisor's 20 philosophies. This reference doesn't
give specific fonts — that's a **brand decision**.

### 3.8 Focus Switch = Background Dimming + Foreground Sharpening + Flash Lead-in

A focus switch is **not just** lowering opacity. The full recipe is:

```js
// Filter combination for non-focus elements
tile.style.filter = `
  brightness(${1 - 0.5 * focusIntensity})
  saturate(${1 - 0.3 * focusIntensity})
  blur(${focusIntensity * 4}px)        // ← key: blur is what truly "recedes" them
`;
tile.style.opacity = 0.4 + 0.6 * (1 - focusIntensity);

// After focus completes, do a 150ms Flash highlight at the focus position to lead the eye back
focusOverlay.animate([
  { background: 'rgba(255,255,255,0.3)' },
  { background: 'rgba(255,255,255,0)' }
], { duration: 150, easing: 'ease-out' });
```

**Why blur is mandatory**: with only opacity + brightness, the out-of-focus elements
remain "sharp" — visually they don't have the effect of "receding into the background".
blur(4-8px) actually pushes non-focus elements one layer back in depth-of-field.

---

## 4. Concrete Motion Techniques (copy-paste ready snippets)

### 4.1 FLIP / Shared Element Transition

A button "expands" into an input field — **not** the button disappears + a new panel appears.
The core is **the same DOM element** transitioning between two states, not two elements
cross-fading.

```jsx
// Using Framer Motion layoutId
<motion.div layoutId="design-button">Design</motion.div>
// ↓ same layoutId after click
<motion.div layoutId="design-button">
  <input placeholder="Describe your design..." />
</motion.div>
```

For native implementation see https://aerotwist.com/blog/flip-your-animations/

### 4.2 "Breathing" Expand (width→height)

Panel expansion is **not pulling width and height simultaneously** — it's:
- First 40% of time: pull only width (keep height small)
- Last 60% of time: hold width, push height

This simulates the physical-world feel of "first unfold, then fill with water".

```js
const widthT = interpolate(t, [0, 0.4], [0, 1], Easing.expoOut);
const heightT = interpolate(t, [0.3, 1], [0, 1], Easing.expoOut);
style.width = `${widthT * targetW}px`;
style.height = `${heightT * targetH}px`;
```

### 4.3 Staggered Fade-up (30ms stagger)

For table rows, card columns, and list item entrances, **delay each element by 30ms**,
with `translateY` going from 10px back to 0.

```js
rows.forEach((row, i) => {
  const localT = Math.max(0, t - i * 0.03);  // 30ms stagger
  row.style.opacity = interpolate(localT, [0, 0.3], [0, 1], Easing.expoOut);
  row.style.transform = `translateY(${
    interpolate(localT, [0, 0.3], [10, 0], Easing.expoOut)
  }px)`;
});
```

### 4.4 Non-linear Breathing · 0.5s Hold Before a Key Result

The machine executes fast and continuously, but **hold for 0.5 seconds before a key result appears**,
giving the viewer's brain reaction time.

```jsx
// Typical scene: AI finishes generating → 0.5s hold → result appears
<Sprite start={8} end={8.5}>
  {/* 0.5s pause — nothing moves, let the viewer fix on the loading state */}
  <LoadingState />
</Sprite>
<Sprite start={8.5} end={10}>
  <ResultAppear />
</Sprite>
```

**Counter-example**: AI finishes generating and seamlessly cuts to the result —
viewer has no reaction time, information is lost.

### 4.5 Chunk Reveal · Simulating Token Streaming

For AI-generated text, **don't use `setInterval` to spit single characters** (like old movie
subtitles); use **chunk reveal** — 2-5 characters at a time, irregular intervals,
simulating real token streaming output.

```js
// Chunk by chunk, not char by char
const chunks = text.split(/(\s+|,\s*|\.\s*|;\s*)/);  // split by word + punctuation
let i = 0;
function reveal() {
  if (i >= chunks.length) return;
  element.textContent += chunks[i++];
  const delay = 40 + Math.random() * 80;  // irregular 40-120ms
  setTimeout(reveal, delay);
}
reveal();
```

### 4.6 Anticipation → Action → Follow-through

3 of Disney's 12 principles. Anthropic uses these very explicitly:

- **Anticipation**: a small reverse motion before the action starts (button slightly shrinks before popping)
- **Action**: the main motion itself
- **Follow-through**: an aftermath after the motion ends (card slightly bounces after landing)

```js
// Full three-stage card entrance
const anticip = interpolate(t, [0, 0.2], [1, 0.95], Easing.easeIn);     // anticipation
const action  = interpolate(t, [0.2, 0.7], [0.95, 1.05], Easing.expoOut); // action
const settle  = interpolate(t, [0.7, 1], [1.05, 1], Easing.spring);       // follow-through
// Final scale = product of three stages or stage-by-stage application
```

**Counter-example**: animations with only Action and no Anticipation + Follow-through
feel like "PowerPoint animations".

### 4.7 3D Perspective + translateZ Layering

For the "tilted 3D + floating cards" feel, give the container perspective and individual
elements different translateZ values:

```css
.stage-wrap {
  perspective: 2400px;
  perspective-origin: 50% 30%;  /* sightline slightly looking down */
}
.card-grid {
  transform-style: preserve-3d;
  transform: rotateX(8deg) rotateY(-4deg);  /* golden ratio */
}
.card:nth-child(3n) { transform: translateZ(30px); }
.card:nth-child(5n) { transform: translateZ(-20px); }
.card:nth-child(7n) { transform: translateZ(60px); }
```

**Why rotateX 8° / rotateY -4° is the golden ratio**:
- More than 10° → element distortion is too strong, looks like "falling over"
- Less than 5° → looks like "skew" rather than "perspective"
- The asymmetric 8° × -4° ratio simulates the natural angle of "the camera looking down from the upper-left of the desktop"

### 4.8 Diagonal Pan · Move XY Simultaneously

Camera motion is not pure up-down or pure left-right — it's **moving XY simultaneously**
to simulate diagonal motion:

```js
const panX = Math.sin(flowT * 0.22) * 40;
const panY = Math.sin(flowT * 0.35) * 30;
stage.style.transform = `
  translate(-50%, -50%)
  rotateX(8deg) rotateY(-4deg)
  translate3d(${panX}px, ${panY}px, 0)
`;
```

**Key**: X and Y have different frequencies (0.22 vs 0.35) to avoid Lissajous loop regularity.

---

## 5. Scene Recipes (Three Narrative Templates)

The three reference videos correspond to three product personalities. **Pick the one that
best fits your product** — don't mix and match.

### Recipe A · Apple Keynote Drama (Claude Design type)

**Suits**: major version launches, hero animations, visual wow first
**Pacing**: Slow-Fast-Boom-Stop strong arc
**Easing**: `expoOut` throughout + a touch of `overshoot`
**SFX density**: high (~0.4/s), SFX pitch tuned to BGM scale
**BGM**: IDM / minimal tech electronica, cool + precise
**Ending**: rapid camera pull-back → drop → Logo morph → ethereal single tone → abrupt stop

### Recipe B · One-Shot Tool Flow (Claude Code type)

**Suits**: developer tools, productivity apps, flow scenes
**Pacing**: sustained steady flow, no obvious peak
**Easing**: `spring` physics + `expoOut`
**SFX density**: **0** (entirely BGM-driven editing rhythm)
**BGM**: Lo-fi Hip-hop / Boom-bap, 85-90 BPM
**Core technique**: key UI actions land on BGM kick/snare transients — "**musical groove as interaction sound effect**"

### Recipe C · Office Productivity Narrative (Claude for Word type)

**Suits**: enterprise software, document/spreadsheet/calendar apps, professional feel first
**Pacing**: multi-scene hard cuts + Dolly In/Out
**Easing**: `overshoot` (toggle) + `expoOut` (panel)
**SFX density**: medium (~0.3/s), UI clicks dominate
**BGM**: Jazzy Instrumental, minor key, BPM 90-95
**Core highlight**: one scene must have the "whole-piece highlight" — 3D pop-out / lifting off the plane

---

## 6. Counter-examples · Doing These Is AI Slop

| Anti-pattern | Why it's wrong | Correct approach |
|---|---|---|
| `transition: all 0.3s ease` | `ease` is a relative of linear, all elements move at the same speed | Use `expoOut` + per-element stagger |
| All entrances are `opacity 0→1` | No sense of motion direction | Pair with `translateY 10→0` + Anticipation |
| Logo fade-in | No narrative collapse | Morph / Converge / collapse-expand |
| Mouse moves in a straight line | Subconscious machine feel | Bézier arc + Perlin Noise |
| Char-by-char typing (setInterval) | Like old movie subtitles | Chunk Reveal, random intervals |
| Key result with no hold | Viewer has no reaction time | 0.5s hold before result |
| Focus switch only changes opacity | Out-of-focus elements stay sharp | opacity + brightness + **blur** |
| Pure black bg / pure white bg | Cyberpunk feel / reflective fatigue | Neutrals with color temperature (per brand spec) |
| All animations equally fast | No rhythm | Slow-Fast-Boom-Stop |
| Fade-out ending | No sense of decision | Abrupt stop (hold the last frame) |

---

## 7. Pre-Delivery Checklist (60 seconds before delivery)

- [ ] Is the narrative structure Slow-Fast-Boom-Stop, not even pacing?
- [ ] Is the default easing `expoOut`, not `easeOut` or `linear`?
- [ ] Toggle / button pop uses `overshoot`?
- [ ] Cards / lists entrance has 30ms stagger?
- [ ] 0.5s hold before key result?
- [ ] Typing uses Chunk Reveal, not setInterval per-char?
- [ ] Focus switch adds blur (not just opacity)?
- [ ] Logo is morph-collapse (Morph), not fade-in?
- [ ] Background is not pure black / pure white (color temperature)?
- [ ] Type has serif + sans-serif hierarchy?
- [ ] Ending is abrupt stop, not fade-out?
- [ ] (If a mouse) is the mouse trail an arc, not a straight line?
- [ ] SFX density matches product personality (see Recipe A/B/C)?
- [ ] BGM and SFX have a 6-8dB loudness gap? (see `audio-design-rules.md`)

---

## 8. Relationship to Other References

| reference | Position | Relationship |
|---|---|---|
| `animation-pitfalls.md` | Technical avoidance (16 items) | "**don't do this**" · this file's opposite |
| `animations.md` | Stage/Sprite engine usage | Foundation for **how to write** animations |
| `audio-design-rules.md` | Dual-track audio rules | Rules for **adding audio** to animation |
| `sfx-library.md` | 37-item SFX list | Sound effect **asset library** |
| `apple-gallery-showcase.md` | Apple gallery showcase style | A topic on a specific motion style |
| **This file** | Positive motion design grammar | "**do this**" |

**Invocation order**:
1. First look at SKILL.md workflow Step 3's positioning four-questions (decides narrative role and visual temperature)
2. After choosing direction, read this file to determine **motion language** (Recipe A/B/C)
3. While coding, refer to `animations.md` and `animation-pitfalls.md`
4. When exporting video, follow `audio-design-rules.md` + `sfx-library.md`

---

## Appendix · Source Material for This File

- Anthropic official animation teardown: ai-design project directory's `reference-animations/BEST-PRACTICES.md`
- Anthropic audio teardown: same directory's `AUDIO-BEST-PRACTICES.md`
- 3 reference videos: `ref-{1,2,3}.mp4` + corresponding `gemini-ref-*.md` / `audio-ref-*.md`
- **Strict filtering**: this reference does not include any specific brand color values, font names, or product names.
  Color/font decisions go through the §1.a Core Asset Protocol or the 20 design philosophies.
