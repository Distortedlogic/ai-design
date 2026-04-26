# Design Context: Start From What Already Exists

**This is the single most important thing about this skill.**

A good hi-fi design always grows out of existing design context. **Doing hi-fi from thin air is a last resort, and will inevitably produce generic work.** So at the start of every design task, ask first: is there anything to reference?

## What Is Design Context

In priority order, high to low:

### 1. The user's design system / UI kit
The user's product already has a component library, color tokens, type specs, and an icon system. **The ideal case.**

### 2. The user's codebase
If the user gives you the codebase, there are live component implementations inside. Read those component files:
- `theme.ts` / `colors.ts` / `tokens.css` / `_variables.scss`
- Concrete components (Button.tsx, Card.tsx)
- Layout scaffold (App.tsx, MainLayout.tsx)
- Global stylesheets

**Read the code and copy exact values**: hex codes, spacing scale, font stack, border radius. Don't redraw from memory.

### 3. The user's shipped product
If the user has a product live but didn't give you code, use Playwright or have the user provide screenshots.

```bash
# Use Playwright to screenshot a public URL
npx playwright screenshot https://example.com screenshot.png --viewport-size=1920,1080
```

This lets you see the actual visual vocabulary.

### 4. Brand guidelines / logo / existing assets
The user may have: logo files, brand color specs, marketing collateral, slide templates. All of this is context.

### 5. Competitor references
If the user says "like XX site" — have them provide the URL or a screenshot. **Don't** work from a blurry impression in your training data.

### 6. A known design system (fallback)
If none of the above exists, use a publicly known design system as a base:
- Apple HIG
- Material Design 3
- Radix Colors (palette)
- shadcn/ui (components)
- Tailwind default palette

Tell the user explicitly what you're using, so they know it's a starting point, not the final.

## How to Acquire Context

### Step 1: Ask the user

The mandatory checklist at task kickoff (from `workflow.md`):

```markdown
1. Do you have an existing design system / UI kit / component library? Where?
2. Are there brand guidelines or color/type specs?
3. Can you give me screenshots or URLs of the existing product?
4. Is there a codebase I can read?
```

### Step 2: When the user says "no," help them find it

Don't just give up. Try:

```markdown
Let me see if there are clues:
- Do any of your previous projects have related designs?
- What colors / type does your company's marketing site use?
- What style is your product's logo? Could you send me one?
- Are there any products you admire that we could reference?
```

### Step 3: Read all the context you can find

If the user gives you a codebase path, read it:
1. **List the file structure first**: find files related to styles / theme / components
2. **Read the theme/token files**: lift the actual hex / px values
3. **Read 2-3 representative components**: see the visual vocabulary (hover state, shadow, border, padding patterns)
4. **Read the global stylesheet**: base reset, font loading
5. **If there's a Figma link / screenshot**: look at the image, but **trust the code more**

**Important**: **don't** glance once and work from impression. You've only really lifted context after pulling 30+ specific values.

### Step 4: Vocalize the system you'll use

After reading the context, tell the user the system you'll use:

```markdown
Based on your codebase and product screenshots, here's the design system I've extracted:

**Color**
- Primary: #C27558 (from tokens.css)
- Background: #FDF9F0
- Text: #1A1A1A
- Muted: #6B6B6B

**Type**
- Display: Instrument Serif (from global.css's @font-face)
- Body: Geist Sans
- Mono: JetBrains Mono

**Spacing** (from your scale system)
- 4, 8, 12, 16, 24, 32, 48, 64

**Shadow patterns**
- `0 1px 2px rgba(0,0,0,0.04)` (subtle card)
- `0 10px 40px rgba(0,0,0,0.1)` (elevated modal)

**Border-radius**
- Small components 4px, cards 12px, buttons 8px

**Component vocabulary**
- Button: filled primary, outlined secondary, ghost tertiary, all 8px rounded
- Card: white background, subtle shadow, no border

I'll start working with this system. Confirm it looks right?
```

Wait for the user's confirmation before moving.

## Designing From Thin Air (Fallback When There's No Context)

**Strong warning**: output quality will drop noticeably in this case. Tell the user explicitly.

```markdown
You don't have design context, so I can only work from generic intuition.
The output will be "looks OK but lacks distinctiveness."
Do you want to proceed, or supply some reference material first?
```

If the user insists you proceed, make decisions in this order:

### 1. Pick an aesthetic direction
Don't deliver a generic result. Pick a clear direction:
- brutally minimal
- editorial / magazine
- brutalist / raw
- organic / natural
- luxury / refined
- playful / toy
- retro-futuristic
- soft / pastel

Tell the user which one you picked.

### 2. Pick a known design system as the skeleton
- Use Radix Colors for palette (https://www.radix-ui.com/colors)
- Use shadcn/ui for component vocabulary (https://ui.shadcn.com)
- Use Tailwind's spacing scale (multiples of 4)

### 3. Pick a font pairing with character

Don't use Inter / Roboto. Suggested combinations (free from Google Fonts):
- Instrument Serif + Geist Sans
- Cormorant Garamond + Inter Tight
- Bricolage Grotesque + Söhne (paid)
- Fraunces + Work Sans (note: Fraunces has been worn out by AI)
- JetBrains Mono + Geist Sans (technical feel)

### 4. Every key decision needs a reason

Don't pick silently. Write it in HTML comments:

```html
<!--
Design decisions:
- Primary color: warm terracotta (oklch 0.65 0.18 25) — fits the "editorial" direction  
- Display: Instrument Serif for humanist, literary feel
- Body: Geist Sans for cleanness contrast
- No gradients — committed to minimal, no AI slop
- Spacing: 8px base, golden ratio friendly (8/13/21/34)
-->
```

## Import Strategy (When the User Gives You a Codebase)

If the user says "import this codebase as reference":

### Small (<50 files)
Read it all. Internalize the context.

### Medium (50-500 files)
Focus on:
- `src/components/` or `components/`
- All style / token / theme related files
- 2-3 representative full-page components (Home.tsx, Dashboard.tsx)

### Large (>500 files)
Have the user point you to a focus:
- "I'm building the settings page" → read existing settings-related files
- "I'm building a new feature" → read the overall shell + the closest reference
- Don't aim for completeness; aim for accuracy.

## Working with Figma / Design Files

If the user gives you a Figma link:

- **Don't** expect to "convert Figma to HTML" directly — that requires extra tooling.
- Figma links are usually not publicly accessible.
- Have the user: **export to a screenshot** and send it + tell you the specific color / spacing values.

If they only give you a Figma screenshot, tell the user:
- I can see the visuals, but I can't extract precise values.
- Please send me the key numbers (hex, px), or use Figma's "export as code" feature.

## Final Reminder

**The ceiling on a project's design quality is set by the quality of the context you receive.**

Spending 10 minutes gathering context is worth more than spending an hour drawing hi-fi from thin air.

**When you encounter a no-context situation, ask the user for context first — don't push through on your own.**
