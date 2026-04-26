# Animations: Timeline Animation Engine

Read this when making animation / motion design HTML. Principles, usage, typical patterns.

## Core Pattern: Stage + Sprite

Our animation system (`assets/animations.jsx`) provides a timeline-driven engine:

- **`<Stage>`**: container for the entire animation; automatically provides auto-scale (fit viewport) + scrubber + play/pause/loop controls
- **`<Sprite start end>`**: a time slice. A Sprite only displays during `start` to `end`. Inside, you can read its local progress `t` (0→1) via the `useSprite()` hook
- **`useTime()`**: read the current global time (seconds)
- **`Easing.easeInOut` / `Easing.easeOut` / ...**: easing functions
- **`interpolate(t, from, to, easing?)`**: interpolate based on t

This pattern borrows from Remotion / After Effects ideas, but is lightweight and zero-dependency.

## Getting started

```html
<script type="text/babel" src="animations.jsx"></script>
<script type="text/babel">
  const { Stage, Sprite, useTime, useSprite, Easing, interpolate } = window.Animations;

  function Title() {
    const { t } = useSprite();  // local progress 0→1
    const opacity = interpolate(t, [0, 1], [0, 1], Easing.easeOut);
    const y = interpolate(t, [0, 1], [40, 0], Easing.easeOut);
    return (
      <h1 style={{ 
        opacity, 
        transform: `translateY(${y}px)`,
        fontSize: 120,
        fontWeight: 900,
      }}>
        Hello.
      </h1>
    );
  }

  function Scene() {
    return (
      <Stage duration={10}>  {/* 10-second animation */}
        <Sprite start={0} end={3}>
          <Title />
        </Sprite>
        <Sprite start={2} end={5}>
          <SubTitle />
        </Sprite>
        {/* ... */}
      </Stage>
    );
  }

  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<Scene />);
</script>
```

## Common animation patterns

### 1. Fade In / Fade Out

```jsx
function FadeIn({ children }) {
  const { t } = useSprite();
  const opacity = interpolate(t, [0, 0.3], [0, 1], Easing.easeOut);
  return <div style={{ opacity }}>{children}</div>;
}
```

**Note on the range**: `[0, 0.3]` means the fade-in completes in the first 30% of the sprite's time, then opacity stays at 1.

### 2. Slide In

```jsx
function SlideIn({ children, from = 'left' }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, 0.4], [0, 1], Easing.easeOut);
  const offset = (1 - progress) * 100;
  const directions = {
    left: `translateX(-${offset}px)`,
    right: `translateX(${offset}px)`,
    top: `translateY(-${offset}px)`,
    bottom: `translateY(${offset}px)`,
  };
  return (
    <div style={{
      transform: directions[from],
      opacity: progress,
    }}>
      {children}
    </div>
  );
}
```

### 3. Character-by-character typewriter

```jsx
function Typewriter({ text }) {
  const { t } = useSprite();
  const charCount = Math.floor(text.length * Math.min(t * 2, 1));
  return <span>{text.slice(0, charCount)}</span>;
}
```

### 4. Number counting

```jsx
function CountUp({ from = 0, to = 100, duration = 0.6 }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, duration], [0, 1], Easing.easeOut);
  const value = Math.floor(from + (to - from) * progress);
  return <span>{value.toLocaleString()}</span>;
}
```

### 5. Segmented explainer (typical instructional animation)

```jsx
function Scene() {
  return (
    <Stage duration={20}>
      {/* Phase 1: present the problem */}
      <Sprite start={0} end={4}>
        <Problem />
      </Sprite>

      {/* Phase 2: present the approach */}
      <Sprite start={4} end={10}>
        <Approach />
      </Sprite>

      {/* Phase 3: present the result */}
      <Sprite start={10} end={16}>
        <Result />
      </Sprite>

      {/* Caption shown throughout */}
      <Sprite start={0} end={20}>
        <Caption />
      </Sprite>
    </Stage>
  );
}
```

## Easing functions

Preset easing curves:

| Easing | Characteristic | Use for |
|--------|------|------|
| `linear` | Constant speed | Scrolling captions, sustained motion |
| `easeIn` | Slow→fast | Exit / disappear |
| `easeOut` | Fast→slow | Entrance / appear |
| `easeInOut` | Slow→fast→slow | Position changes |
| **`expoOut`** ⭐ | **Exponential ease-out** | **Anthropic-grade primary easing** (physical heft) |
| **`overshoot`** ⭐ | **Springy bounce-back** | **Toggle / button pop / emphasized interaction** |
| `spring` | Spring physics | Interaction feedback, geometry settling |
| `anticipation` | First reverse, then forward | Emphasized motion |

**Default primary easing is `expoOut`** (not `easeOut`) — see `animation-best-practices.md` §2.
Use `expoOut` for entrance, `easeIn` for exit, `overshoot` for toggle — the foundational rules of Anthropic-grade animation.

## Pacing and duration guide

### Micro-interactions (0.1-0.3s)
- Button hover
- Card expand
- Tooltip appear

### UI transitions (0.3-0.8s)
- Page switch
- Modal appear
- List item join

### Narrative animations (2-10s per segment)
- One phase of a concept explanation
- Reveal of a data chart
- Scene transition

### A single narrative segment is at most 10 seconds
Human attention is finite. Tell one thing in 10 seconds, then move on.

## Order of thinking when designing animation

### 1. Content/story first, animation second

**Wrong**: want fancy animations first, then stuff content in
**Right**: clearly figure out what to communicate first, then use animation as a means to serve that information

Animation is **signal**, not **decoration**. A fade-in emphasizes "this is important, look here" — if everything fades in, the signal stops working.

### 2. Write the timeline by Scene

```
0:00 - 0:03   Problem appears (fade in)
0:03 - 0:06   Problem zoomed/expanded (zoom+pan)
0:06 - 0:09   Solution appears (slide in from right)
0:09 - 0:12   Solution explained (typewriter)
0:12 - 0:15   Result demoed (counter up + chart reveal)
0:15 - 0:18   Summary line (static, read for 3s)
0:18 - 0:20   CTA or fade out
```

Write the timeline first, then write the components.

### 3. Assets first

Images/icons/fonts the animation will use should be **prepared first**. Don't go searching for assets halfway through drawing — it breaks rhythm.

## Common issues

**Animation stutters**
→ Mostly layout thrashing. Use `transform` and `opacity`; don't animate `top`/`left`/`width`/`height`/`margin`. The browser GPU-accelerates `transform`.

**Animation too fast, can't see clearly**
→ A human reads one Chinese character in 100-150ms, one word in 300-500ms. If you're telling a story with text, leave at least 3 seconds per sentence.

**Animation too slow, viewers bored**
→ Interesting visual changes should be dense. A static frame held over 5 seconds gets dull.

**Multiple animations interfere with each other**
→ Use CSS `will-change: transform` to tell the browser ahead of time that this element will move, reducing reflow.

**Recording into a video**
→ Use the toolchain bundled with the skill (one command outputs three formats): see `video-export.md`
- `scripts/render-video.js` — HTML → 25fps MP4 (Playwright + ffmpeg)
- `scripts/convert-formats.sh` — 25fps MP4 → 60fps MP4 + optimized GIF
- Want more precise frame rendering? Make render(t) a pure function; see `animation-pitfalls.md` rule 5

## Pairing with video tools

This skill makes **HTML animation** (running in the browser). If the final output is to be used as video footage:

- **Short animations / concept demo**: use methods here for HTML animation → screen recording
- **Long videos / narrative**: this skill focuses on HTML animation; for long videos use AI video generation skills or professional video software
- **Motion graphics**: professional After Effects / Motion Canvas is more suitable

## On Popmotion and similar libraries

If you really need physical animation (spring, decay, keyframes with precise timing) that our engine can't handle, you can fall back to Popmotion:

```html
<script src="https://unpkg.com/popmotion@11.0.5/dist/popmotion.min.js"></script>
```

But **try our engine first**. It handles 90% of cases.
