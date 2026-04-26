# Tweaks: Live Parameter Tuning for Design Variations

Tweaks is one of the core capabilities of this skill — letting the user switch variations and tune parameters live without editing code.

**Cross-agent compatibility**: Some native design-agent environments (like Claude.ai Artifacts) rely on the host's postMessage to write tweak values back into the source for persistence. This skill uses a **pure-frontend localStorage approach** — the effect is identical (state survives refresh), but persistence happens in the browser's localStorage instead of the source file. This approach works in any agent environment (Claude Code / Codex / Cursor / Trae / etc.).

## When to Add Tweaks

- The user explicitly asks for "tunable parameters" / "switching between versions"
- The design has multiple variations that benefit from comparison
- The user didn't say so, but you judge that **a few well-chosen tweaks would help them see the possibility space**

Default recommendation: **add 2–3 tweaks to every design** (color theme / font size / layout variant) even if the user didn't ask — letting them see the possibility space is part of the design service.

## Implementation (pure-frontend version)

### Basic structure

```jsx
const TWEAK_DEFAULTS = {
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
};

function useTweaks() {
  const [tweaks, setTweaks] = React.useState(() => {
    try {
      const stored = localStorage.getItem('design-tweaks');
      return stored ? { ...TWEAK_DEFAULTS, ...JSON.parse(stored) } : TWEAK_DEFAULTS;
    } catch {
      return TWEAK_DEFAULTS;
    }
  });

  const update = (patch) => {
    const next = { ...tweaks, ...patch };
    setTweaks(next);
    try {
      localStorage.setItem('design-tweaks', JSON.stringify(next));
    } catch {}
  };

  const reset = () => {
    setTweaks(TWEAK_DEFAULTS);
    try {
      localStorage.removeItem('design-tweaks');
    } catch {}
  };

  return { tweaks, update, reset };
}
```

### Tweaks panel UI

Floating panel in the bottom-right. Collapsible:

```jsx
function TweaksPanel() {
  const { tweaks, update, reset } = useTweaks();
  const [open, setOpen] = React.useState(false);

  return (
    <div style={{
      position: 'fixed',
      bottom: 20,
      right: 20,
      zIndex: 9999,
    }}>
      {open ? (
        <div style={{
          background: 'white',
          border: '1px solid #e5e5e5',
          borderRadius: 12,
          padding: 20,
          boxShadow: '0 10px 40px rgba(0,0,0,0.12)',
          width: 280,
          fontFamily: 'system-ui',
          fontSize: 13,
        }}>
          <div style={{ 
            display: 'flex', 
            justifyContent: 'space-between', 
            alignItems: 'center',
            marginBottom: 16,
          }}>
            <strong>Tweaks</strong>
            <button onClick={() => setOpen(false)} style={{
              border: 'none', background: 'none', cursor: 'pointer', fontSize: 16,
            }}>×</button>
          </div>

          {/* Color */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Primary</div>
            <input 
              type="color" 
              value={tweaks.primaryColor} 
              onChange={e => update({ primaryColor: e.target.value })}
              style={{ width: '100%', height: 32 }}
            />
          </label>

          {/* Font size slider */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Font size ({tweaks.fontSize}px)</div>
            <input 
              type="range" 
              min={12} max={24} step={1}
              value={tweaks.fontSize}
              onChange={e => update({ fontSize: +e.target.value })}
              style={{ width: '100%' }}
            />
          </label>

          {/* Density select */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Density</div>
            <select 
              value={tweaks.density}
              onChange={e => update({ density: e.target.value })}
              style={{ width: '100%', padding: 6 }}
            >
              <option value="compact">Compact</option>
              <option value="comfortable">Comfortable</option>
              <option value="spacious">Spacious</option>
            </select>
          </label>

          {/* Dark mode toggle */}
          <label style={{ 
            display: 'flex', 
            alignItems: 'center',
            gap: 8,
            marginBottom: 16,
          }}>
            <input 
              type="checkbox" 
              checked={tweaks.dark}
              onChange={e => update({ dark: e.target.checked })}
            />
            <span>Dark mode</span>
          </label>

          <button onClick={reset} style={{
            width: '100%',
            padding: '8px 12px',
            background: '#f5f5f5',
            border: 'none',
            borderRadius: 6,
            cursor: 'pointer',
            fontSize: 12,
          }}>Reset</button>
        </div>
      ) : (
        <button 
          onClick={() => setOpen(true)}
          style={{
            background: '#1A1A1A',
            color: 'white',
            border: 'none',
            borderRadius: 999,
            padding: '10px 16px',
            fontSize: 12,
            cursor: 'pointer',
            boxShadow: '0 4px 12px rgba(0,0,0,0.15)',
          }}
        >⚙ Tweaks</button>
      )}
    </div>
  );
}
```

### Applying Tweaks

Use Tweaks in the main component:

```jsx
function App() {
  const { tweaks } = useTweaks();

  return (
    <div style={{
      '--primary': tweaks.primaryColor,
      '--font-size': `${tweaks.fontSize}px`,
      background: tweaks.dark ? '#0A0A0A' : '#FAFAFA',
      color: tweaks.dark ? '#FAFAFA' : '#1A1A1A',
    }}>
      {/* your content */}
      <TweaksPanel />
    </div>
  );
}
```

Use the variables in CSS:

```css
button.cta {
  background: var(--primary);
  color: white;
  font-size: var(--font-size);
}
```

## Typical Tweak Options

What tweaks to add for different design types:

### Universal
- Primary color (color picker)
- Font size (slider 12–24px)
- Typeface (select: display font vs body font)
- Dark mode (toggle)

### Slide deck
- Theme (light / dark / brand)
- Background style (solid / gradient / image)
- Type contrast (more decorative vs more restrained)
- Information density (minimal / standard / dense)

### Product prototype
- Layout variant (layout A / B / C)
- Interaction speed (animation speed 0.5x–2x)
- Data volume (mock data rows: 5 / 20 / 100)
- State (empty / loading / success / error)

### Animation
- Speed (0.5x–2x)
- Loop (once / loop / ping-pong)
- Easing (linear / easeOut / spring)

### Landing page
- Hero style (image / gradient / pattern / solid)
- CTA copy (a few variants)
- Structure (single column / two column / sidebar)

## Tweaks Design Principles

### 1. Meaningful options, not busywork

Every tweak must surface a **real design choice**. Don't add tweaks no one would ever switch (e.g. a 0–50px border-radius slider — the user discovers every middle value is ugly).

A good tweak exposes **discrete, considered variations**:
- "Corner style": none / subtle / bold (three options)
- Not: "Corner radius": 0–50px slider

### 2. Less is more

A design's Tweaks panel has **at most 5–6** options. More than that and it turns into a "configuration page" and loses the point of fast variation exploration.

### 3. Defaults are the finished design

Tweaks are **icing on the cake**. The default values must themselves be a complete, shippable design. What the user sees with the Tweaks panel closed is the deliverable.

### 4. Group sensibly

When there are many options, group them visually:

```
---- Visual ----
Primary | Font size | Dark mode

---- Layout ----
Density | Sidebar position

---- Content ----
Data volume | State
```

## Forward Compatibility with Source-level Persistence Hosts

If you later want the design to also work on environments that support source-level tweaks (like Claude.ai Artifacts), keep the **EDITMODE marker block**:

```jsx
const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
}/*EDITMODE-END*/;
```

The marker block is **inert** in the localStorage scheme (just a regular comment), but on hosts that support source rewriting it gets read for source-level persistence. Adding it has no downside in the current environment and keeps you forward-compatible.

## Common Issues

**The Tweaks panel covers the design content**
→ Make it closeable. Closed by default, with a small button the user clicks to expand.

**The user has to reset tweaks every time they refresh**
→ It's already using localStorage. If it doesn't persist after refresh, check whether localStorage is available (incognito mode fails — wrap in catch).

**Multiple HTML pages should share tweaks**
→ Namespace the localStorage key with the project name: `design-tweaks-[projectName]`.

**I want tweaks to interact with each other**
→ Add the logic in `update`:

```jsx
const update = (patch) => {
  let next = { ...tweaks, ...patch };
  // Linkage: when dark mode is selected, auto-switch text color
  if (patch.dark === true && !patch.textColor) {
    next.textColor = '#F0EEE6';
  }
  setTweaks(next);
  localStorage.setItem(...);
};
```
