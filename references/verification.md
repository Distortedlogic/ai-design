# Verification: Output Validation Flow

Some native design-agent environments (like Claude.ai Artifacts) have a built-in `fork_verifier_agent` that spins up a subagent to screenshot and check via iframe. Most agent environments (Claude Code / Codex / Cursor / Trae / etc.) don't have this — using Playwright manually covers the same verification scenarios.

## Verification Checklist

Each time you produce HTML, run this checklist:

### 1. Browser render check (mandatory)

Most basic: **does the HTML even open**? On macOS:

```bash
open -a "Google Chrome" "/path/to/your/design.html"
```

Or use Playwright screenshots (next section).

### 2. Console error check

The most common problem in HTML files is JS errors causing a white screen. Run with Playwright:

```bash
python ~/.claude/skills/claude-design/scripts/verify.py path/to/design.html
```

This script:
1. Opens the HTML in headless Chromium
2. Saves a screenshot to the project directory
3. Captures console errors
4. Reports status

See `scripts/verify.py` for details.

### 3. Multi-viewport check

For responsive designs, capture multiple viewports:

```bash
python verify.py design.html --viewports 1920x1080,1440x900,768x1024,375x667
```

### 4. Interaction check

Tweaks, animations, button toggles — default static screenshots can't see them. **Best to have the user click through it themselves in a browser**, or record video with Playwright:

```python
page.video.record('interaction.mp4')
```

### 5. Per-slide check for slide decks

Deck-style HTML, capture one slide at a time:

```bash
python verify.py deck.html --slides 10  # capture the first 10 slides
```

Generates `deck-slide-01.png`, `deck-slide-02.png`, ... for quick review.

## Playwright Setup

First-time setup:

```bash
# If not yet installed
npm install -g playwright
npx playwright install chromium

# Or the Python version
pip install playwright
playwright install chromium
```

If the user already has Playwright installed globally, just use it.

## Screenshot Best Practices

### Capture full page

```python
page.screenshot(path='full.png', full_page=True)
```

### Capture viewport only

```python
page.screenshot(path='viewport.png')  # default: visible area only
```

### Capture a specific element

```python
element = page.query_selector('.hero-section')
element.screenshot(path='hero.png')
```

### High-DPI screenshot

```python
page = browser.new_page(device_scale_factor=2)  # retina
```

### Wait for animations to finish before capturing

```python
page.wait_for_timeout(2000)  # wait 2s for the animation to settle
page.screenshot(...)
```

## Sending Screenshots to the User

### Open local screenshots directly

```bash
open screenshot.png
```

The user views them in their own Preview / Figma / VSCode / browser.

### Upload to image host and share a link

If you need to share with remote collaborators (Slack, Feishu, WeChat, etc.), have the user upload via their own image host tool or MCP:

```bash
python ~/Documents/writing/tools/upload_image.py screenshot.png
```

Returns an ImgBB permanent link you can paste anywhere.

## When Verification Fails

### White screen

There's definitely a console error. Check first:

1. The integrity hash on the React + Babel script tag is correct (see `react-setup.md`)
2. Naming conflict on `const styles = {...}`
3. Cross-file components missing an export to `window`
4. JSX syntax errors (babel.min.js doesn't report; switch to the unminified babel.js)

### Janky animation

- Record a session with Chrome DevTools' Performance tab
- Look for layout thrashing (frequent reflow)
- Prefer `transform` and `opacity` for animation (GPU accelerated)

### Wrong fonts

- Check that `@font-face` URLs are reachable
- Check the fallback fonts
- Chinese fonts load slowly: show the fallback first, then swap once loaded

### Layout breaks

- Check that `box-sizing: border-box` is applied globally
- Check the `* margin: 0; padding: 0` reset
- Turn on gridlines in Chrome DevTools to see the actual layout

## Verification = the Designer's Second Set of Eyes

**Always go through it yourself**. When AI writes code, you regularly see:

- Looks correct but the interaction has bugs
- Static screenshot looks fine but it breaks on scroll
- Looks great wide but collapses on narrow screens
- Forgot to test dark mode
- Some components don't respond after Tweaks toggle

**The last 1 minute of verification can save 1 hour of rework.**

## Common Verification Script Commands

```bash
# Basic: open + screenshot + capture errors
python verify.py design.html

# Multi-viewport
python verify.py design.html --viewports 1920x1080,375x667

# Multi-slide
python verify.py deck.html --slides 10

# Output to a specific directory
python verify.py design.html --output ./screenshots/

# headless=false, open a real browser so you can watch
python verify.py design.html --show
```
