# Video Export: HTML Animation to MP4/GIF

Once an animation HTML is done, users often ask "can you export it to video?" This guide gives the full flow.

## When to Export

**Right time to export**:
- The animation runs end-to-end and is visually verified (Playwright screenshots confirm correct state at each timepoint)
- The user has watched it in a browser at least once and signed off on the effect
- **Don't** export while there are still animation bugs — fixing them after exporting to video is more expensive

**Trigger phrases the user might say**:
- "Can you export it to video?"
- "Convert to MP4"
- "Make a GIF"
- "60fps"

## Output Specs

By default, deliver three formats and let the user choose:

| Format | Spec | Use case | Typical size (30s) |
|---|---|---|---|
| MP4 25fps | 1920×1080 · H.264 · CRF 18 | WeChat article embed, Channels, YouTube | 1–2 MB |
| MP4 60fps | 1920×1080 · minterpolate frame interpolation · H.264 · CRF 18 | High-frame-rate showcase, Bilibili, portfolio | 1.5–3 MB |
| GIF | 960×540 · 15fps · palette-optimized | Twitter/X, README, Slack preview | 2–4 MB |

## Toolchain

Two scripts in `scripts/`:

### 1. `render-video.js` — HTML → MP4

Records a 25fps MP4 base version. Depends on global Playwright.

```bash
NODE_PATH=$(npm root -g) node /path/to/claude-design/scripts/render-video.js <html-file>
```

Optional flags:
- `--duration=30` animation length (seconds)
- `--width=1920 --height=1080` resolution
- `--trim=2.2` seconds to trim from the start (removes reload + font load time)
- `--fontwait=1.5` font load wait (seconds); raise this when there are many fonts

Output: same directory as the HTML, with the same name and `.mp4` extension.

### 2. `add-music.sh` — MP4 + BGM → MP4

Mixes background music into a silent MP4, picking from the built-in BGM library by mood, or supply your own audio. Auto-matches duration and adds fade in/out.

```bash
bash add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
```

**Built-in BGM library** (in `assets/bgm-<mood>.mp3`):

| `--mood=` | Style | Suitable for |
|-----------|------|---------|
| `tech` (default) | Apple Silicon / Apple keynote vibe, minimal synth + piano | Product launch, AI tools, skill promo |
| `ad` | Upbeat modern electronic, with build + drop | Social ads, product teaser, promo |
| `educational` | Warm and bright, soft guitar / electric piano, inviting | Explainers, tutorial intros, course teasers |
| `educational-alt` | Same family, alternate track | Same as above |
| `tutorial` | Lo-fi ambient, almost no presence | Software demos, coding tutorials, long demos |
| `tutorial-alt` | Same family, alternate track | Same as above |

**Behavior**:
- Music is trimmed to video length
- 0.3s fade-in + 1s fade-out (avoids hard cuts)
- Video stream `-c:v copy` (no re-encode), audio AAC 192k
- `--music=<path>` takes priority over `--mood`; you can directly pass any external audio
- Wrong mood name lists all valid options instead of failing silently

**Typical pipeline** (animation export trio + BGM):
```bash
node render-video.js animation.html                        # record
bash convert-formats.sh animation.mp4                      # derive 60fps + GIF
bash add-music.sh animation-60fps.mp4                      # add default tech BGM
# Or for different scenarios:
bash add-music.sh tutorial-demo.mp4 --mood=tutorial
bash add-music.sh product-promo.mp4 --mood=ad --out=promo-final.mp4
```

### 3. `convert-formats.sh` — MP4 → 60fps MP4 + GIF

Generate a 60fps version and a GIF from an existing MP4.

```bash
bash /path/to/claude-design/scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
```

Output (same directory as input):
- `<name>-60fps.mp4` — defaults to `fps=60` frame duplication (broad compatibility); add `--minterpolate` to enable high-quality interpolation
- `<name>.gif` — palette-optimized GIF (default 960 wide; configurable)

**Choosing 60fps mode**:

| Mode | Command | Compatibility | When to use |
|---|---|---|---|
| Frame duplication (default) | `convert-formats.sh in.mp4` | QuickTime / Safari / Chrome / VLC all play | General delivery, upload platforms, social media |
| minterpolate interpolation | `convert-formats.sh in.mp4 --minterpolate` | macOS QuickTime / Safari may refuse to open | Bilibili and other showcase scenarios that need true interpolation; **always test the target player locally before delivery** |

Why is the default now frame duplication? minterpolate output's H.264 elementary stream has a known compat bug — when minterpolate was the default, we hit "macOS QuickTime won't open it" multiple times. See `animation-pitfalls.md` §14.

`gif_width` parameter:
- 960 (default) — universal for social platforms
- 1280 — sharper but bigger file
- 600 — Twitter/X priority loading

## Full Flow (recommended standard)

After the user says "export to video":

```bash
cd <project-directory>

# Assume $SKILL points to this skill's root (replace with your install path)

# 1. Record the 25fps base MP4
NODE_PATH=$(npm root -g) node "$SKILL/scripts/render-video.js" my-animation.html

# 2. Derive the 60fps MP4 and GIF
bash "$SKILL/scripts/convert-formats.sh" my-animation.mp4

# Output:
# my-animation.mp4         (25fps · 1-2 MB)
# my-animation-60fps.mp4   (60fps · 1.5-3 MB)
# my-animation.gif         (15fps · 2-4 MB)
```

## Technical Details (for debugging)

### Playwright recordVideo gotchas

- Frame rate is fixed at 25fps; you can't directly record at 60fps (Chromium headless compositor limit)
- Recording starts at context creation, so you must `trim` away the leading load time
- Default format is webm; needs ffmpeg to transcode to H.264 MP4 for universal playback

`render-video.js` already handles the above.

### ffmpeg minterpolate parameters

Current config: `minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1`

- `mi_mode=mci` — motion compensation interpolation
- `mc_mode=aobmc` — adaptive overlapped block motion compensation
- `me_mode=bidir` — bidirectional motion estimation
- `vsbmc=1` — variable-size block motion compensation

Works well on CSS **transform animations** (translate/scale/rotate).
On **pure fades** it can produce slight ghosting — if the user dislikes it, fall back to simple frame duplication:

```bash
ffmpeg -i input.mp4 -r 60 -c:v libx264 ... output.mp4
```

### Why GIF palette is two-pass

GIF supports only 256 colors. A single-pass GIF compresses the whole animation into one generic 256-color palette, which muddies subtle palettes (like beige + orange).

Two-pass:
1. `palettegen=stats_mode=diff` — first scan the whole clip and generate **the optimal palette for this animation**
2. `paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle` — encode using that palette; rectangle diff only updates changed regions, drastically shrinking the file

For fade transitions, `dither=bayer` is smoother than `none` but produces a slightly larger file.

## Pre-flight Check (before exporting)

A 30-second self-check before exporting:

- [ ] HTML runs end-to-end in a browser with no console errors
- [ ] Frame 0 of the animation is the full initial state (not blank/loading)
- [ ] The last frame of the animation is a stable ending state (not a half-cut)
- [ ] Fonts / images / emoji all render correctly (see `animation-pitfalls.md`)
- [ ] The duration parameter matches the actual animation length in the HTML
- [ ] In the HTML, Stage detection of `window.__recording` forces `loop=false` (must check on hand-written Stages; built into `assets/animations.jsx`)
- [ ] Final Sprite has `fadeOut={0}` (don't fade the last frame of the video)
- [ ] Includes a "Created by ai-design" watermark (mandatory only for animation pieces; for third-party brand work, prefix with "Unofficial · ". See SKILL.md §"Skill Promo Watermark" for details)

## Notes to Include with Delivery

Standard format for the notes you give the user after export:

```
**Full delivery**

| File | Format | Spec | Size |
|---|---|---|---|
| foo.mp4 | MP4 | 1920×1080 · 25fps · H.264 | X MB |
| foo-60fps.mp4 | MP4 | 1920×1080 · 60fps (motion interpolation) · H.264 | X MB |
| foo.gif | GIF | 960×540 · 15fps · palette-optimized | X MB |

**Notes**
- 60fps uses minterpolate for motion-estimation interpolation; works well on transform animations
- GIF is palette-optimized; a 30s animation typically compresses to around 3MB

Tell me if you want a different size or frame rate.
```

## Common Follow-up Requests

| User says | Response |
|---|---|
| "Too big" | MP4: raise CRF to 23–28. GIF: drop resolution to 600 or fps to 10 |
| "GIF is fuzzy" | Raise `gif_width` to 1280; or suggest using MP4 instead (WeChat Moments supports it too) |
| "I want vertical 9:16" | Change the HTML source to `--width=1080 --height=1920` and re-record |
| "Add a watermark" | ffmpeg `-vf "drawtext=..."` or `overlay=` a PNG |
| "I want a transparent background" | MP4 doesn't support alpha; use WebM VP9 + alpha or APNG |
| "I want lossless" | CRF 0 + preset veryslow (file gets ~10× larger) |
