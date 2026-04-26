# Content Guidelines: Avoiding AI Slop, Content Principles, Scale Specs

These are the easiest traps to fall into when doing AI design. This is a "what NOT to do" list, and that matters more than a "what to do" list — because AI slop is the default. If you don't actively avoid it, it will happen.

## The Complete AI Slop Blacklist

### Visual Traps

**❌ Aggressive gradient backgrounds**
- Purple → pink → blue full-screen gradients (the signature flavor of AI-generated webpages)
- Rainbow gradients in any direction
- Mesh gradients tiled across the background
- ✅ If you must use a gradient: keep it subtle, monochromatic, and intentional (e.g., a button hover state)

**❌ Rounded cards with a left-border accent color**
```css
/* This is the signature of AI-flavored cards */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```
This card style is everywhere in AI-generated dashboards. Want emphasis? Use a more designerly approach: background contrast, weight/size contrast, plain dividers, or just don't separate things into cards.

**❌ Emoji decoration**
Unless the brand itself uses emoji (e.g., Notion, Slack), don't put emoji in the UI. **Especially avoid**:
- 🚀 ⚡️ ✨ 🎯 💡 in front of titles
- ✅ in feature lists
- → emoji in CTA buttons (a standalone arrow glyph is fine; an emoji arrow is not)

If you don't have an icon, use a real icon library (Lucide / Heroicons / Phosphor), or use a placeholder.

**❌ Drawing imagery with SVG**
Don't try to draw with SVG: people, scenes, devices, objects, abstract art. AI-drawn SVG imagery screams "AI" — it's childish and cheap. **A gray rectangle with the label "illustration placeholder 1200×800" beats a clumsy SVG hero illustration by a factor of 100.**

The only situations where SVG is acceptable:
- Real icons (16×16 to 32×32 range)
- Geometric shapes as decorative elements
- Data viz charts

**❌ Too much iconography**
Not every title / feature / section needs an icon. Overusing icons makes the interface look like a toy. Less is more.

**❌ "Data slop"**
Made-up stats used as decoration:
- "10,000+ happy customers" (you don't even know if this is true)
- "99.9% uptime" (don't write it without real data)
- Decorative "metric cards" cobbled from icon + number + phrase
- Mock tables dressed up with fake data

If you don't have real data, leave a placeholder or ask the user for it.

**❌ "Quote slop"**
Made-up user testimonials and famous-person quotes used to decorate the page. Leave a placeholder and ask the user for real quotes.

### Type Traps

**❌ Avoid these overused fonts**:
- Inter (the default for AI-generated webpages)
- Roboto
- Arial / Helvetica
- The bare system font stack
- Fraunces (AI discovered this and ran it into the ground)
- Space Grotesk (AI's recent favorite)

**✅ Use a display + body pairing with character**. Direction inspiration:
- Serif display + sans-serif body (editorial feel)
- Mono display + sans body (technical feel)
- Heavy display + light body (contrast)
- Variable font for hero weight animation

Type resources:
- Underused good options on Google Fonts (Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono)
- Open-source font sites (Fraunces' siblings, Adobe Fonts)
- Don't invent font names from thin air

### Color Traps

**❌ Inventing colors from thin air**
Don't design an entire unfamiliar palette from scratch. It usually doesn't harmonize.

**✅ Strategy**:
1. Brand color exists → use the brand color, fill missing tokens by interpolating in oklch
2. No brand color but a reference exists → eyedrop from screenshots of the reference product
3. Completely from scratch → pick a known color system (Radix Colors / Tailwind default palette / Anthropic brand) — don't tune your own.

**Defining colors in oklch** is the most modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* lighter shade, same hue */
  --primary-dark: oklch(0.45 0.20 25);  /* darker shade, same hue */
}
```
oklch guarantees the hue doesn't drift when adjusting brightness — better than hsl.

**❌ Throwing inverted colors at dark mode**
It's not a simple invert. A good dark mode requires re-tuned saturation, contrast, and accent colors. If you don't want to do dark mode properly, don't do it.

### Layout Traps

**❌ Bento grid overuse**
Every AI-generated landing page wants to do bento. Unless your information structure genuinely fits bento, use a different layout.

**❌ Big hero + 3-column features + testimonials + CTA**
This landing-page template is exhausted. If you want to be original, actually be original.

**❌ Card grid where every card looks identical**
Asymmetric cards, varied sizes, some with images and some text-only, some spanning columns — that's what real designers' work looks like.

## Content Principles

### 1. Don't add filler content

Every element must earn its place. Whitespace is a design problem to be solved by **composition** (contrast, rhythm, breathing room), **not** by filling it with content.

**The filler test**:
- If you remove this content, does the design get worse? If the answer is "no," remove it.
- What real problem does this element solve? If the answer is "to make the page feel less empty," delete it.
- Is there real data backing this stat / quote / feature? If not, don't make it up.

"One thousand no's for every yes."

### 2. Ask before adding material

You think adding another section / page / paragraph would be better? Ask the user first; don't add it unilaterally.

Reasons:
- The user knows their audience better than you do
- Adding content has a cost; the user may not want it
- Adding content unilaterally violates the "junior designer reporting to a lead" relationship

### 3. Create a system up front

Once you've explored the design context, **state the system you plan to use out loud** and let the user confirm:

```markdown
My design system:
- Color: #1A1A1A as the primary text + #F0EEE6 background + #D97757 accent (from your brand)
- Type: Instrument Serif for display + Geist Sans for body
- Rhythm: section titles use a full-bleed colored background + white text; regular sections use white backgrounds
- Imagery: hero uses a full-bleed photo; feature sections use placeholders pending your assets
- At most 2 background colors to avoid clutter

Confirm this direction and I'll start building.
```

Wait for confirmation before acting. This check-in prevents the "halfway through and the direction is wrong" outcome.

## Scale Specs

### Slides (1920×1080)

- Body text minimum **24px**, ideal 28-36px
- Titles 60-120px
- Section titles 80-160px
- Hero headlines can be 180-240px
- Never use fonts <24px on a slide

### Print documents

- Body text minimum **10pt** (≈13.3px), ideal 11-12pt
- Titles 18-36pt
- Captions 8-9pt

### Web and mobile

- Body text minimum **14px** (16px for senior-friendly designs)
- Mobile body **16px** (avoids iOS auto-zoom)
- Hit targets (clickable elements) minimum **44×44px**
- Line height 1.5-1.7 (Chinese 1.7-1.8)

### Contrast

- Body text vs background **at least 4.5:1** (WCAG AA)
- Large text vs background **at least 3:1**
- Verify with Chrome DevTools' accessibility tools

## CSS Power Tools

**Advanced CSS features** are a designer's best friend — use them boldly:

### Typography

```css
/* Make headline line breaks more natural; avoids a single orphaned word on the last line */
h1, h2, h3 { text-wrap: balance; }

/* Body text wrapping that avoids widows and orphans */
p { text-wrap: pretty; }

/* Chinese typography power tools: punctuation kerning, line-start/end control */
p { 
  text-spacing-trim: space-all;
  hanging-punctuation: first;
}
```

### Layout

```css
/* CSS Grid + named areas = readability through the roof */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
}

/* Subgrid aligns content across cards */
.card { display: grid; grid-template-rows: subgrid; }
```

### Visual Effects

```css
/* A scrollbar with design taste */
* { scrollbar-width: thin; scrollbar-color: #666 transparent; }

/* Glassmorphism (use sparingly) */
.glass {
  backdrop-filter: blur(20px) saturate(150%);
  background: color-mix(in oklch, white 70%, transparent);
}

/* The View Transitions API makes page transitions silky */
@view-transition { navigation: auto; }
```

### Interactions

```css
/* The :has() selector makes conditional styling easy */
.card:has(img) { padding-top: 0; } /* cards with images get no top padding */

/* container queries make components truly responsive */
@container (min-width: 500px) { ... }

/* The new color-mix function */
.button:hover {
  background: color-mix(in oklch, var(--primary) 85%, black);
}
```

## Decision Quick Reference: When You're Hesitating

- Want to add a gradient? → Probably don't.
- Want to add an emoji? → Don't.
- Want rounded cards with a border-left accent? → Don't — use another emphasis approach.
- Want to draw an SVG hero illustration? → Don't — use a placeholder.
- Want to add a decorative quote? → First ask the user if they have a real quote.
- Want a row of icon features? → First ask whether icons are needed; probably not.
- Using Inter? → Pick something with more character.
- Using a purple gradient? → Pick a palette with a real reason behind it.

**When you feel "adding this would look nicer" — that's usually the symptom of AI slop**. Ship the simplest version first, and only add when the user asks.
