# Audio Design Rules · ai-design

> Audio application recipes for all animation demos. Use alongside `sfx-library.md` (asset list).
> Battle-tested: ai-design release hero v1-v9 iterations · Gemini deep teardowns of Anthropic's three official pieces · 8000+ A/B comparisons

---

## Core principle · Dual-track audio (iron rule)

Animation audio **must be designed in two independent layers** — you cannot just do one:

| Layer | Function | Time scale | Relationship to visuals | Frequency band occupied |
|---|---|---|---|---|
| **SFX (beat layer)** | Mark each visual beat | 0.2-2 seconds short | **Strong sync** (frame-aligned) | **High freq 800Hz+** |
| **BGM (ambience bed)** | Emotional bed, soundscape | Continuous 20-60 seconds | Weak sync (segment-level) | **Mid-low freq <4kHz** |

**Animations with only BGM are crippled** — viewers subconsciously perceive that "things are moving on screen but there's no sound responding"; that's the root of the cheapness.

---

## Gold standard · Golden ratios

These numbers are **engineering hard parameters** distilled from measuring Anthropic's three official pieces + comparing against our own v9 final cut. Apply directly:

### Volume
- **BGM volume**: `0.40-0.50` (relative to full scale 1.0)
- **SFX volume**: `1.00`
- **Loudness gap**: BGM is **-6 to -8 dB lower** than SFX peak (not by SFX absolute loudness standing out, but by the loudness gap)
- **amix parameter**: `normalize=0` (never use normalize=1; it flattens the dynamic range)

### Frequency-band isolation (P1 hard optimization)
Anthropic's secret isn't "loud SFX volume" — it's **frequency-band layering**:

```bash
[bgm_raw]lowpass=f=4000[bgm]      # BGM limited to mid-low freq <4kHz
[sfx_raw]highpass=f=800[sfx]      # SFX pushed up to mid-high freq 800Hz+
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]
```

Why: the human ear is most sensitive in the 2-5kHz range (the "presence" band). If SFX all sit in that band and BGM covers the full spectrum, **SFX gets masked by BGM's high-frequency components**. Using highpass to push SFX up + lowpass to push BGM down lets the two each hold their own zone in the spectrum, and SFX clarity goes up a notch.

### Fade
- BGM in: `afade=in:st=0:d=0.3` (0.3s, avoids hard cut)
- BGM out: `afade=out:st=N-1.5:d=1.5` (1.5s long tail, sense of closure)
- SFX has its own envelope; no extra fade needed

---

## SFX cue design rules

### Density (how many SFX per 10 seconds)
Measured SFX density in Anthropic's three pieces falls into three tiers:

| Piece | SFX per 10s | Product personality | Scene |
|---|---|---|---|
| Artifacts (ref-1) | **~9 / 10s** | Feature-dense, info-heavy | Complex tool demo |
| Code Desktop (ref-2) | **0** | Pure ambience, meditative | Dev tool focus state |
| Word (ref-3) | **~4 / 10s** | Balanced, office tempo | Productivity tool |

**Heuristic**:
- Cool / focused product personality → low SFX density (0-3 / 10s), BGM-led
- Lively / info-heavy product personality → high SFX density (6-9 / 10s), SFX drives pacing
- **Don't fill every visual beat** — restraint is more premium than density. **Cutting 30-50% of the cues makes the rest more dramatic**.

### Cue selection priority
Not every visual beat needs SFX. Pick by this priority:

**P0 mandatory** (omitting feels off):
- Typing (terminal / input)
- Click / select (user decision moment)
- Focus switch (visual subject shift)
- Logo reveal (brand collapse)

**P1 recommended**:
- Element entrance / exit (modal / card)
- Completion / success feedback
- AI generation start / end
- Major transition (scene switch)

**P2 optional** (too many gets messy):
- hover / focus-in
- progress tick
- decorative ambient

### Timestamp alignment precision
- **Same-frame alignment** (0ms error): click / focus switch / Logo settle
- **1-2 frames early** (-33ms): rapid whoosh (gives viewer a psychological lead-in)
- **1-2 frames late** (+33ms): object landing / impact (matches real physics)

---

## BGM selection decision tree

The ai-design skill ships with 6 BGM tracks (`assets/bgm-*.mp3`):

```
What's the animation personality?
├─ Product launch / tech demo → bgm-tech.mp3 (minimal synth + piano)
├─ Tutorial / tool usage → bgm-tutorial.mp3 (warm, instructional)
├─ Education / principle explanation → bgm-educational.mp3 (curious, thoughtful)
├─ Marketing ad / brand promo → bgm-ad.mp3 (upbeat, promotional)
└─ Same style needs a variant → bgm-*-alt.mp3 (alternate version of each)
```

### No-BGM scenarios (worth considering)
Reference Anthropic Code Desktop (ref-2): **0 SFX + pure Lo-fi BGM** can also be very premium.

**When to choose no-BGM**:
- Animation length <10s (BGM can't get established)
- Product personality is "focus / meditation"
- Scene already has ambient sound / narration
- SFX density is very high (avoid auditory overload)

---

## Scene recipes (out of the box)

### Recipe A · Product launch hero (same as ai-design v9)
```
Length: 25 seconds
BGM: bgm-tech.mp3 · 45% · band <4kHz
SFX density: ~6 / 10s

cues:
  Terminal typing → type × 4 (0.6s gaps)
  Enter           → enter
  Card converge   → card × 4 (staggered 0.2s)
  Selected        → click
  Ripple          → whoosh
  4 focus switches → focus × 4
  Logo            → thud (1.5s)

Volume: BGM 0.45 / SFX 1.0 · amix normalize=0
```

### Recipe B · Tool feature demo (reference Anthropic Code Desktop)
```
Length: 30-45 seconds
BGM: bgm-tutorial.mp3 · 50%
SFX density: 0-2 / 10s (very few)

Strategy: let BGM + voiceover narration drive; SFX only at **decisive moments** (file save / command execution complete)
```

### Recipe C · AI generation demo
```
Length: 15-20 seconds
BGM: bgm-tech.mp3 or no BGM
SFX density: ~8 / 10s (high density)

cues:
  User input → type + enter
  AI starts processing → magic/ai-process (1.2s loop)
  Generation complete → feedback/complete-done
  Result reveal → magic/sparkle
  
Highlight: ai-process can loop 2-3 times to span the entire generation process
```

### Recipe D · Pure-ambience long shot (reference Artifacts)
```
Length: 10-15 seconds
BGM: none
SFX: 3-5 carefully designed cues used standalone

Strategy: every SFX is the protagonist; no "muddied with BGM" issue.
Suits: single-product slow shot, close-up showcase
```

---

## ffmpeg composition templates

### Template 1 · Single SFX layered onto video
```bash
ffmpeg -y -i video.mp4 -itsoffset 2.5 -i sfx.mp3 \
  -filter_complex "[0:a][1:a]amix=inputs=2:normalize=0[a]" \
  -map 0:v -map "[a]" output.mp4
```

### Template 2 · Multi-SFX timeline composition (aligned by cue time)
```bash
ffmpeg -y \
  -i sfx-type.mp3 -i sfx-enter.mp3 -i sfx-click.mp3 -i sfx-thud.mp3 \
  -filter_complex "\
[0:a]adelay=1100|1100[a0];\
[1:a]adelay=3200|3200[a1];\
[2:a]adelay=7000|7000[a2];\
[3:a]adelay=21800|21800[a3];\
[a0][a1][a2][a3]amix=inputs=4:duration=longest:normalize=0[mixed]" \
  -map "[mixed]" -t 25 sfx-track.mp3
```
**Key parameters**:
- `adelay=N|N`: first is left-channel delay (ms), second is right; write twice to ensure stereo alignment
- `normalize=0`: preserve dynamic range — critical!
- `-t 25`: truncate to specified length

### Template 3 · Video + SFX track + BGM (with frequency-band isolation)
```bash
ffmpeg -y -i video.mp4 -i sfx-track.mp3 -i bgm.mp3 \
  -filter_complex "\
[2:a]atrim=0:25,afade=in:st=0:d=0.3,afade=out:st=23.5:d=1.5,\
     lowpass=f=4000,volume=0.45[bgm];\
[1:a]highpass=f=800,volume=1.0[sfx];\
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k final.mp4
```

---

## Failure-mode quick reference

| Symptom | Root cause | Fix |
|---|---|---|
| SFX inaudible | BGM high-freq component masks it | Add `lowpass=f=4000` to BGM + `highpass=f=800` to SFX |
| SFX too loud, harsh | SFX absolute volume too high | Drop SFX volume to 0.7 and BGM to 0.3, preserving the gap |
| BGM and SFX rhythms clash | Wrong BGM (used music with strong beat) | Switch to ambient / minimal synth BGM |
| BGM cuts off abruptly at animation end | No fade out | `afade=out:st=N-1.5:d=1.5` |
| SFX overlap into mush | Cues too dense + each SFX too long | Keep SFX length under 0.5s, cue gap ≥ 0.2s |
| WeChat public-account mp4 has no sound | Public account sometimes mutes auto-play | Don't worry; users get sound on click; gif has no sound by nature |

---

## Coupling with visuals (advanced)

### SFX timbre should match visual style
- Warm rice / paper-feel visuals → SFX with **wood / soft** timbre (Morse, paper snap, soft click)
- Cold black-tech visuals → SFX with **metal / digital** timbre (beep, pulse, glitch)
- Hand-drawn / playful visuals → SFX with **cartoon / exaggerated** timbre (boing, pop, zap)

Our current `apple-gallery-showcase.md` warm-rice background → pair with `keyboard/type.mp3` (mechanical) + `container/card-snap.mp3` (soft) + `impact/logo-reveal-v2.mp3` (cinematic bass)

### SFX can lead visual pacing
Advanced technique: **design the SFX timeline first, then adjust the visual animation to align with the SFX** (not the other way around).
Because every SFX cue is a "clock tick", visual animation adapting to SFX rhythm is very stable — conversely, SFX chasing visuals often feels off when ±1 frame is off.

---

## Quality check list (self-check before release)

- [ ] Loudness gap: SFX peak - BGM peak = -6 to -8 dB?
- [ ] Frequency: BGM lowpass 4kHz + SFX highpass 800Hz?
- [ ] amix normalize=0 (preserve dynamic range)?
- [ ] BGM fade-in 0.3s + fade-out 1.5s?
- [ ] SFX count appropriate (pick density by scene personality)?
- [ ] Each SFX aligned same-frame with visual beat (within ±1 frame)?
- [ ] Logo reveal SFX long enough (suggested 1.5s)?
- [ ] Listen with BGM off: do the SFX alone have enough rhythm?
- [ ] Listen with SFX off: does BGM alone have emotional arc?

Either layer alone should be coherent. If only the two stacked together sound good, you didn't do it right.

---

## References

- SFX asset list: `sfx-library.md`
- Visual style reference: `apple-gallery-showcase.md`
- Anthropic three pieces' deep audio analysis: `/Users/alchain/Documents/Writing/01-Public Account Writing/Project/2026.04-ai-design release/Reference Animation/AUDIO-BEST-PRACTICES.md`
- ai-design v9 production case: `/Users/alchain/Documents/Writing/01-Public Account Writing/Project/2026.04-ai-design release/Artwork/hero-animation-v9-final.mp4`
