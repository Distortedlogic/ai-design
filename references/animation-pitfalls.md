# Animation Pitfalls: Bugs and Rules from HTML Animation

The most common bugs hit when making animations and how to avoid them. Every rule comes from a real failure case.

Reading this before writing animation can save a round of iteration.

## 1. Stacking layout — `position: relative` is the default obligation

**The pitfall**: a sentence-wrap element wraps 3 bracket-layers (`position: absolute`). Without setting `position: relative` on sentence-wrap, the absolute brackets use `.canvas` as their coordinate system and fly 200px below the screen.

**Rules**:
- Any container holding `position: absolute` children **must** explicitly have `position: relative`
- Even when you don't visually need any "offset", write `position: relative` as a coordinate-system anchor
- When you write `.parent { ... }` whose children include `.child { position: absolute }`, instinctively give parent relative

**Quick check**: every time `position: absolute` appears, count up the ancestors and make sure the nearest positioned ancestor is the coordinate system you *want*.

## 2. Character traps — don't rely on rare Unicode

**The pitfall**: tried to use `␣` (U+2423 OPEN BOX) to visualize a "space token". Neither Noto Serif SC nor Cormorant Garamond has this glyph; it renders as blank/tofu and viewers see nothing.

**Rules**:
- **Every character that appears in animation must exist in the font you chose**
- Common rare-character blacklist: `␣ ␀ ␐ ␋ ␨ ↩ ⏎ ⌘ ⌥ ⌃ ⇧ ␦ ␖ ␛`
- To convey meta-characters like "space / return / tab", use **CSS-built semantic boxes**:
  ```html
  <span class="space-key">Space</span>
  ```
  ```css
  .space-key {
    display: inline-flex;
    padding: 4px 14px;
    border: 1.5px solid var(--accent);
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.3em;
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }
  ```
- Emojis need verifying too: some emojis fall back to gray boxes outside Noto Emoji; better to use an `emoji` font-family or SVG

## 3. Data-driven Grid/Flex templates

**The pitfall**: code has `const N = 6` tokens, but CSS is hard-coded as `grid-template-columns: 80px repeat(5, 1fr)`. The 6th token has no column and the entire matrix is misaligned.

**Rules**:
- When count comes from a JS array (`TOKENS.length`), the CSS template should also be data-driven
- Option A: inject from JS via CSS variables
  ```js
  el.style.setProperty('--cols', N);
  ```
  ```css
  .grid { grid-template-columns: 80px repeat(var(--cols), 1fr); }
  ```
- Option B: use `grid-auto-flow: column` and let the browser auto-extend
- **Forbid the "fixed number + JS constant" combination** — when N changes, CSS won't sync

## 4. Transition gaps — scene switches must be continuous

**The pitfall**: between zoom1 (13-19s) → zoom2 (19.2-23s), the main sentence is already hidden, zoom1 fade out (0.6s) + zoom2 fade in (0.6s) + stagger delay (0.2s+) = about 1 second of pure blank screen. Viewers think the animation has frozen.

**Rules**:
- When switching scenes consecutively, fade out and fade in should **cross-overlap**, not "previous one fully gone before the next begins"
  ```js
  // Bad:
  if (t >= 19) hideZoom('zoom1');      // 19.0s out
  if (t >= 19.4) showZoom('zoom2');    // 19.4s in → 0.4s gap in the middle

  // Good:
  if (t >= 18.6) hideZoom('zoom1');    // start fading out 0.4s early
  if (t >= 18.6) showZoom('zoom2');    // fade in at the same time (cross-fade)
  ```
- Or use an "anchor element" (e.g. the main sentence) as a visual link between scenes; have it briefly reappear during zoom switches
- Compute the CSS transition duration carefully so the next transition doesn't fire before the current one ends

## 5. Pure Render principle — animation state should be seekable

**The pitfall**: using `setTimeout` + `fireOnce(key, fn)` to chain-trigger animation state. Plays back fine normally, but during frame-by-frame recording or seeking to arbitrary timestamps, the previously-fired setTimeouts cannot "go back in time".

**Rules**:
- The `render(t)` function should ideally be a **pure function**: given t, it outputs a unique DOM state
- If side effects are unavoidable (e.g. class toggling), use a `fired` set with explicit reset:
  ```js
  const fired = new Set();
  function fireOnce(key, fn) { if (!fired.has(key)) { fired.add(key); fn(); } }
  function reset() { fired.clear(); /* clear all .show classes */ }
  ```
- Expose `window.__seek(t)` for Playwright / debugging:
  ```js
  window.__seek = (t) => { reset(); render(t); };
  ```
- Animation-related setTimeouts should not span >1 second; otherwise seek-rewind goes haywire

## 6. Measuring before fonts load = measuring wrong

**The pitfall**: at DOMContentLoaded, page calls `charRect(idx)` to measure bracket position; fonts haven't loaded, so each character's width is the fallback font's width and positions are all wrong. After fonts load (~500ms later), the bracket's `left: Xpx` is still the old value, permanently offset.

**Rules**:
- Any layout code dependent on DOM measurements (`getBoundingClientRect`, `offsetWidth`) **must** be wrapped inside `document.fonts.ready.then()`
  ```js
  document.fonts.ready.then(() => {
    requestAnimationFrame(() => {
      buildBrackets(...);  // fonts are now ready, measurement is accurate
      tick();              // animation begins
    });
  });
  ```
- The extra `requestAnimationFrame` gives the browser one frame to commit layout
- If using Google Fonts CDN, `<link rel="preconnect">` speeds up the first load

## 7. Recording prep — leave handles for video export

**The pitfall**: Playwright `recordVideo` defaults to 25fps and starts recording the moment the context is created. The first 2 seconds of page load and font loading get recorded. Delivered video has 2 seconds of blank/white-flash at the front.

**Rules**:
- Provide a `render-video.js` tool that handles: warmup navigate → reload restarts the animation → wait duration → ffmpeg trim head + transcode H.264 MP4
- The animation's **frame 0** should be the complete initial state with final layout in place (not blank or loading)
- Want 60fps? Use ffmpeg `minterpolate` for post-processing; don't expect it from the browser source frame rate
- Want a GIF? Two-stage palette (`palettegen` + `paletteuse`); a 30s 1080p animation can compress to 3MB

See `video-export.md` for full script invocation.

## 8. Batch export — tmp directories must include PID to prevent concurrent collisions

**The pitfall**: ran 3 parallel `render-video.js` processes recording 3 HTMLs. Because TMP_DIR was named with only `Date.now()`, when 3 processes started in the same millisecond they shared the same tmp directory. The first one to finish cleaned up tmp; the other two hit `ENOENT` reading the directory and all crashed.

**Rules**:
- Any temporary directory potentially shared by multiple processes must be named with a **PID or random suffix**:
  ```js
  const TMP_DIR = path.join(DIR, '.video-tmp-' + Date.now() + '-' + process.pid);
  ```
- If you really want multi-file parallelism, use shell `&` + `wait` rather than forking inside one node script
- For batch recording multiple HTMLs, the conservative approach: **serial** execution (up to 2 in parallel; 3+ should queue up)

## 9. Recording captures progress bars / replay buttons — Chrome elements pollute the video

**The pitfall**: animation HTML has a `.progress` bar, `.replay` button, and `.counter` timestamp added for human debugging. When recorded into the delivered MP4, these elements appear at the bottom of the video as if devtools had been screen-recorded.

**Rules**:
- Separate management for "chrome elements" meant for humans (progress bar / replay button / footer / masthead / counter / phase labels) and the actual video content
- **Convention class name** `.no-record`: any element with this class is auto-hidden by the recording script
- Script side (`render-video.js`) injects CSS by default to hide common chrome class names:
  ```
  .progress .counter .phases .replay .masthead .footer .no-record [data-role="chrome"]
  ```
- Inject via Playwright `addInitScript` (takes effect before each navigate, robust on reload)
- Add `--keep-chrome` flag to view the original HTML (with chrome)

## 10. First few seconds of recording show animation repeating — Warmup frame leak

**The pitfall**: old `render-video.js` flow `goto → wait fonts 1.5s → reload → wait duration`. Recording starts the moment context is created; during warmup the animation has already played some, then reload restarts from 0. The result: the first few seconds of video show "mid-animation + cut + animation from 0", with strong duplication feel.

**Rules**:
- **Warmup and Record must use independent contexts**:
  - Warmup context (no `recordVideo` option): only loads url, waits for fonts, then closes
  - Record context (with `recordVideo`): starts fresh, animation records from t=0
- ffmpeg `-ss trim` can only trim Playwright's tiny startup latency (~0.3s); it **cannot** mask warmup frames; the source must be clean
- Closing the record context = WebM file written to disk; this is a Playwright constraint
- Related code pattern:
  ```js
  // Phase 1: warmup (throwaway)
  const warmupCtx = await browser.newContext({ viewport });
  const warmupPage = await warmupCtx.newPage();
  await warmupPage.goto(url, { waitUntil: 'networkidle' });
  await warmupPage.waitForTimeout(1200);
  await warmupCtx.close();

  // Phase 2: record (fresh)
  const recordCtx = await browser.newContext({ viewport, recordVideo });
  const page = await recordCtx.newPage();
  await page.goto(url, { waitUntil: 'networkidle' });
  await page.waitForTimeout(DURATION * 1000);
  await page.close();
  await recordCtx.close();
  ```

## 11. Don't draw "fake chrome" inside the canvas — decorative player UI collides with real chrome

**The pitfall**: animation uses the `Stage` component, which already provides a scrubber + timecode + pause button (`.no-record` chrome, auto-hidden on export). I then drew a "magazine-page-style decorative progress bar" `00:60 ──── CLAUDE-DESIGN / ANATOMY` at the bottom and felt good about it. **Result**: the user saw two progress bars — one is the Stage controller, one is my decoration. Visually they completely collide and get flagged as a bug. "What's the deal with another progress bar inside the video?"

**Rules**:

- Stage already provides: scrubber + timecode + pause/replay button. **Do not draw additional** progress indicators, current timecode, copyright signature bars, or chapter counters inside the canvas — they either collide with chrome, or they're filler slop (violates the "earn its place" principle).
- "Page-number feel", "magazine feel", "bottom signature bar" — these **decorative urges** are the high-frequency filler AI auto-adds. Be alert every time one appears: does it really convey irreplaceable information, or is it just filling empty space?
- If you firmly believe a bottom strip must exist (e.g. the animation theme is about player UI), it must be **narratively necessary** AND **visually distinct from the Stage scrubber** (different position, different form, different color tone).

**Element-attribution test** (every element drawn into the canvas must answer):

| What it belongs to | Treatment |
|------------|------|
| Narrative content of a specific scene | OK, keep it |
| Global chrome (control / debug use) | Add `.no-record` class, hidden on export |
| **Belongs to no scene and isn't chrome** | **Delete**. This is an orphan; it must be filler slop |

**Self-check (3 seconds before delivery)**: snap a static frame and ask yourself —

- Is there anything in the frame that "looks like video player UI" (horizontal progress bar, timecode, control button shape)?
- If so, would deleting it harm the narrative? If no harm, delete it.
- Has the same kind of information (progress / time / signature) appeared twice? Consolidate to a single chrome location.

**Counter-examples**: drawing `00:42 ──── PROJECT NAME` at the bottom, drawing "CH 03 / 06" chapter counter in the lower-right, drawing version number "v0.3.1" at the canvas edge — all are fake-chrome filler.

## 12. Recording leading blank + recording start offset — `__ready` × tick × lastTick triple trap

**The pitfall (A · leading blank)**: 60-second animation exported to MP4, the first 2-3 seconds are a blank page. `ffmpeg --trim=0.3` can't cut it.

**The pitfall (B · start offset, real incident on 2026-04-20)**: exported a 24-second video; the user perceived "the video doesn't start playing the first frame until second 19". In fact the animation started recording at t=5, recorded until t=24 then looped back to t=0, then recorded another 5 seconds to end — so the last 5 seconds of the video are actually the beginning of the animation.

**Root cause** (both pitfalls share one root cause):

Playwright `recordVideo` starts writing WebM the moment `newContext()` is called; at that point Babel/React/font loading takes L seconds (2-6s). The recording script waits for `window.__ready = true` as the anchor point for "the animation begins here" — it must be strictly paired with animation `time = 0`. Two common errors:

| Error | Symptom |
|------|------|
| `__ready` set in `useEffect` or sync setup phase (before tick's first frame) | Recording script thinks animation has started; in reality WebM is still recording the blank page → **leading blank** |
| `lastTick = performance.now()` initialized at **script top level** | Font loading L seconds is counted into the first-frame `dt`, `time` instantly jumps to L → recording is L seconds behind throughout → **start offset** |

**✅ Correct full starter tick template** (hand-written animations must use this skeleton):

```js
// ━━━━━━ state ━━━━━━
let time = 0;
let playing = false;   // ❗ default not playing, wait until fonts ready to start
let lastTick = null;   // ❗ sentinel — first tick frame forces dt to 0 (don't use performance.now())
const fired = new Set();

// ━━━━━━ tick ━━━━━━
function tick(now) {
  if (lastTick === null) {
    lastTick = now;
    window.__ready = true;   // ✅ pair: "recording start" is the same frame as "animation t=0"
    render(0);               // render once more to ensure DOM is ready (fonts already ready by now)
    requestAnimationFrame(tick);
    return;
  }
  const dt = (now - lastTick) / 1000;   // dt only starts advancing after the first frame
  lastTick = now;

  if (playing) {
    let t = time + dt;
    if (t >= DURATION) {
      t = window.__recording ? DURATION - 0.001 : 0;  // don't loop while recording, leave 0.001s to keep the last frame
      if (!window.__recording) fired.clear();
    }
    time = t;
    render(time);
  }
  requestAnimationFrame(tick);
}

// ━━━━━━ boot ━━━━━━
// Don't immediately rAF at top level — start only after fonts are loaded
document.fonts.ready.then(() => {
  render(0);                 // first paint the initial frame (fonts ready)
  playing = true;
  requestAnimationFrame(tick);  // first tick will pair __ready + t=0
});

// ━━━━━━ seek interface (for render-video defensive correction) ━━━━━━
window.__seek = (t) => { fired.clear(); time = t; lastTick = null; render(t); };
```

**Why this template is right**:

| Step | Why it must be this way |
|------|-------------|
| `lastTick = null` + first-frame `return` | Avoids counting the "script load → first tick execution" L seconds into animation time |
| `playing = false` by default | While fonts are loading, even if `tick` runs it doesn't advance time, avoiding render mis-alignment |
| `__ready` set on tick first frame | Recording script starts its clock here; the corresponding frame is the animation's true t=0 |
| Start tick only inside `document.fonts.ready.then(...)` | Avoids font fallback width measurement; avoids font-jump on first frame |
| `window.__seek` exists | Lets `render-video.js` actively correct — second line of defense |

**Corresponding defenses on the recording-script side**:
1. `addInitScript` injects `window.__recording = true` (before page goto)
2. `waitForFunction(() => window.__ready === true)`, record the offset at this moment as ffmpeg trim
3. **Extra**: after `__ready`, actively `page.evaluate(() => window.__seek && window.__seek(0))` to force any HTML time drift to zero — second line of defense, against HTML that doesn't strictly follow the starter template

**Verification method**: after exporting MP4
```bash
ffmpeg -i video.mp4 -ss 0 -vframes 1 frame-0.png
ffmpeg -i video.mp4 -ss $DURATION-0.1 -vframes 1 frame-end.png
```
The first frame must be the animation's t=0 initial state (not mid-animation, not black); the last frame must be the animation's final state (not some moment in a second loop).

**Reference implementation**: `assets/animations.jsx`'s Stage component and `scripts/render-video.js` are both implemented to this protocol. Hand-written HTML must wrap the starter tick template — every line is defense against a specific bug.

## 13. No looping while recording — `window.__recording` signal

**The pitfall**: animation Stage defaults to `loop=true` (convenient for viewing in browser). `render-video.js` waits an extra 300ms buffer after recording duration seconds before stopping, and these 300ms put Stage into the next loop. When ffmpeg `-t DURATION` clips, the last 0.5-1s falls into the next loop — the video ending suddenly returns to the first frame (Scene 1) and viewers think the video has a bug.

**Root cause**: there's no "I'm recording" handshake between the recording script and HTML. The HTML doesn't know it's being recorded and continues to loop as in browser interactive scenarios.

**Rules**:

1. **Recording script**: in `addInitScript` inject `window.__recording = true` (before page goto):
   ```js
   await recordCtx.addInitScript(() => { window.__recording = true; });
   ```

2. **Stage component**: recognize this signal and force loop=false:
   ```js
   const effectiveLoop = (typeof window !== 'undefined' && window.__recording) ? false : loop;
   // ...
   if (next >= duration) return effectiveLoop ? 0 : duration - 0.001;
   //                                                       ↑ keep 0.001 to prevent Sprite end=duration being turned off
   ```

3. **Ending Sprite's fadeOut**: in recording scenarios should be set to `fadeOut={0}`, otherwise the video ending will fade to transparent/dark — users expect to stop on a clear last frame, not fade out. When hand-writing HTML, recommend ending Sprites all use `fadeOut={0}`.

**Reference implementation**: `assets/animations.jsx` Stage / `scripts/render-video.js` both have built-in handshakes. Hand-written Stages must implement `__recording` detection — otherwise recording will hit this pitfall.

**Verification**: after exporting MP4 `ffmpeg -ss 19.8 -i video.mp4 -frames:v 1 end.png`, check whether the last 0.2 seconds is still the expected last frame, with no sudden cut to another scene.

## 14. 60fps video defaults to frame duplication — minterpolate has poor compatibility

**The pitfall**: 60fps MP4 generated by `convert-formats.sh` using `minterpolate=fps=60:mi_mode=mci...` cannot open in some macOS QuickTime / Safari versions (black screen or refuses to open). VLC / Chrome can open it.

**Root cause**: the H.264 elementary stream output by minterpolate contains certain SEI / SPS fields that some players have trouble parsing.

**Rules**:

- 60fps default uses simple `fps=60` filter (frame duplication), broad compatibility (QuickTime/Safari/Chrome/VLC can all open)
- High-quality interpolation uses the `--minterpolate` flag explicitly — but **must be tested locally** on the target player before delivery
- The 60fps tag's value is **upload platform algorithmic recognition** (Bilibili / YouTube prioritize 60fps marked uploads); actual perceived smoothness improvement for CSS animations is minimal
- Add `-profile:v high -level 4.0` to improve H.264 universal compatibility

**`convert-formats.sh` already defaults to compatibility mode**. If you need high-quality interpolation, add the `--minterpolate` flag:
```bash
bash convert-formats.sh input.mp4 --minterpolate
```

## 15. `file://` + external `.jsx` CORS trap — single-file delivery must inline the engine

**The pitfall**: animation HTML uses `<script type="text/babel" src="animations.jsx"></script>` to externally load the engine. Double-clicked locally (`file://` protocol) → Babel Standalone XHRs the `.jsx` → Chrome reports `Cross origin requests are only supported for protocol schemes: http, https, chrome, chrome-extension...` → entire page is black; doesn't trigger `pageerror`, only console error, very easy to misdiagnose as "animation didn't fire".

Starting an HTTP server may not save you — when local has a global proxy, even `localhost` goes through the proxy and returns 502 / connection failed.

**Rules**:

- **Single-file delivery (HTML usable by double-click)** → `animations.jsx` must be **inlined** inside `<script type="text/babel">...</script>` tags; don't use `src="animations.jsx"`
- **Multi-file project (HTTP server demo)** → external loading is fine, but on delivery clearly write the `python3 -m http.server 8000` command
- Decision criterion: is what you're delivering to the user "an HTML file" or "a project directory with a server"? The former uses inlining
- Stage component / animations.jsx is often 200+ lines — pasting into the HTML `<script>` block is completely acceptable; don't worry about size

**Minimum verification**: double-click the HTML you generated, **don't** open via any server. Stage should display the animation's first frame correctly to count as passing.

## 16. Cross-scene contrast context — don't hard-code colors on in-canvas elements

**The pitfall**: when making multi-scene animations, elements that **appear across scenes** like `ChapterLabel` / `SceneNumber` / `Watermark` have hard-coded `color: '#1A1A1A'` (dark text) in the component. Fine on the first 4 light-background scenes; on the 5th black-background scene the "05" and watermark disappear — no error, no check triggered, key information invisible.

**Rules**:

- **In-canvas elements reused across multiple scenes** (chapter label / scene number / timecode / watermark / copyright bar) **must not hard-code color values**
- Use one of three approaches instead:
  1. **`currentColor` inheritance**: element only writes `color: currentColor`, parent scene container sets `color: <computed value>`
  2. **invert prop**: component accepts `<ChapterLabel invert />` for manual light/dark switching
  3. **Auto-compute from background**: `color: contrast-color(var(--scene-bg))` (CSS 4 new API, or JS judgment)
- Before delivery use Playwright to grab **a representative frame from each scene** and eyeball whether the "cross-scene elements" are all visible

The insidiousness of this pitfall is — **no bug alarm goes off**. Only human eyes or OCR can catch it.

## Quick self-check list (5 seconds before starting)

- [ ] Every `position: absolute` parent element has `position: relative`?
- [ ] Special characters in animation (`␣` `⌘` `emoji`) all exist in the font?
- [ ] Grid/Flex template count matches JS data length?
- [ ] Cross-fade between scene switches, no >0.3s pure blank?
- [ ] DOM measurement code wrapped in `document.fonts.ready.then()`?
- [ ] `render(t)` is pure, or has explicit reset mechanism?
- [ ] Frame 0 is the complete initial state, not blank?
- [ ] No "fake chrome" decoration in canvas (progress bar / timecode / bottom signature bar colliding with Stage scrubber)?
- [ ] Animation tick first frame synchronously sets `window.__ready = true`? (built into animations.jsx; add it yourself for hand-written HTML)
- [ ] Stage detects `window.__recording` and forces loop=false? (must add for hand-written HTML)
- [ ] Ending Sprite's `fadeOut` set to 0 (video ending stops on clear frame)?
- [ ] 60fps MP4 defaults to frame-duplication mode (compatibility); only add `--minterpolate` for high-quality interpolation?
- [ ] After export, grab frame 0 + final frame to verify they're the animation's initial / final state?
- [ ] Involving specific brands (Stripe/Anthropic/Lovart/...): completed the "Brand Asset Protocol" (SKILL.md §1.a five steps)? Wrote `brand-spec.md`?
- [ ] Single-file delivery HTML: `animations.jsx` is inlined, not `src="..."`? (external .jsx under file:// causes CORS black screen)
- [ ] Cross-scene elements (chapter label / watermark / scene number) have no hard-coded colors? Visible against every scene's background?
