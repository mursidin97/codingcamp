# Design Document — Light Night

## Overview

Light Night is a single-page interactive web application delivered as a **single self-contained HTML file** with no build step, no backend, and no external JavaScript or CSS files beyond a Tailwind CDN script tag and a Google Fonts link tag. The user experience centers on a stylized minimalist character rendered in inline SVG. Clicking or tapping the character (or the CTA button) triggers a coordinated interaction sequence: the counter increments, the mood label changes, a reaction quote fades in, the character bounces and glows, and particle effects burst outward. The entire implementation — markup, styles, and scripts — lives in one file and runs in any modern browser that supports ES6 and CSS3.

**Technology constraints:**
- HTML5 with inline `<style>` and `<script>` blocks
- Tailwind CSS via official CDN `<script>` tag (JIT browser build)
- Google Font (Manrope) via `<link rel="preconnect">` + `<link rel="stylesheet">` in `<head>`
- Vanilla JavaScript (ES6+) — no frameworks, no bundlers, no external JS files
- All animations driven by CSS `transform` and `opacity` only (no layout-reflow properties)

---

## Architecture

The app is a static document with a single responsibility separation pattern achieved entirely within one HTML file:

```
┌─────────────────────────────────────────────────┐
│                  light-night.html                │
│                                                  │
│  ┌──────────┐  ┌────────────┐  ┌─────────────┐  │
│  │  <head>  │  │   <body>   │  │  <script>   │  │
│  │          │  │            │  │             │  │
│  │ Tailwind │  │  DOM Tree  │  │  AppState   │  │
│  │ CDN tag  │  │ (markup +  │  │  + Event    │  │
│  │ Font link│  │  SVG char) │  │  Handlers   │  │
│  │ <style>  │  │            │  │             │  │
│  │ (custom  │  │            │  │             │  │
│  │  CSS +   │  │            │  │             │  │
│  │ keyframes│  │            │  │             │  │
│  │  + vars) │  │            │  │             │  │
│  └──────────┘  └────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────┘
```

### Runtime Data Flow

```
User Input (click / tap / keyboard)
        │
        ▼
handleInteraction()
  ├── Guard: isAnimating? → early return
  ├── State mutation: count++, selectMood(), selectQuote()
  ├── DOM updates: counter, mood label, reaction text
  └── Effect triggers (parallel, non-blocking):
        ├── triggerScaleAnimation()    → CSS class add/remove
        ├── triggerGlowAnimation()     → CSS class add/remove
        ├── triggerCharacterActive()   → CSS class add/remove
        └── spawnParticles(n)          → createElement loop + setTimeout cleanup
```

All side effects are fire-and-forget; the animation lock (`isAnimating`) is released after the Scale Animation duration (400 ms), which is the shortest-lived primary effect.

---

## Components and Interfaces

### DOM Structure

```
<body>                          ← fixed bg gradient via CSS custom props
  #app                          ← flex column, min-h-screen, items-center, justify-center
    header                      ← heading "LIGHT NIGHT" + subtitle
    #stats-bar                  ← flex row, gap, always horizontal
      #counter-component        ← Glassmorphism_Container
        span "INTERACTIONS"
        #counter-value          ← aria-live="polite", displays Interaction_Counter
      #status-display           ← Glassmorphism_Container
        span "MOOD"
        #mood-label             ← CSS transition on text change
    #character-container        ← relative, glass circle, float animation
      #glow-ring                ← absolute, behind char, pulse-glow animation
      #character-svg            ← inline SVG, role="button", tabindex="0"
    #reaction-area              ← Glassmorphism_Container, aria-live="polite"
    #cta-button                 ← <button>, glass style, keyboard accessible
  #particle-container           ← fixed, inset-0, pointer-events-none, z-50
```

### Component Interfaces

#### AppState (JavaScript object)

```js
const state = {
  interactionCount: 0,       // integer ≥ 0
  currentMood: 'CALM',       // string from MOOD_LABELS
  currentQuote: null,        // string from QUOTES or null (before first interaction)
  isAnimating: false         // boolean — debounce lock
};
```

#### handleInteraction() — Public entry point

Called by: character click, character keydown (Enter/Space), CTA button click.

```
handleInteraction() → void
  Pre-condition:  state.isAnimating === false
  Post-condition: state.interactionCount incremented by 1
                  state.currentMood updated (≠ previous)
                  state.currentQuote updated (≠ previous, or first selection)
                  DOM reflects new state
                  Animation effects fired
```

#### selectDifferent(pool, current) — Pure helper

```
selectDifferent(pool: string[], current: string | null) → string
  Pre-condition:  pool.length ≥ 1
  Post-condition: returns a string from pool
                  IF pool.length > 1 THEN result ≠ current
```

#### spawnParticles(count) → void

```
spawnParticles(count: number) → void
  Pre-condition:  count ∈ [6, 12]
  Post-condition: count new .particle elements appended to #particle-container
                  each element removed from DOM after 1200ms + 100ms buffer
                  total .particle count never exceeds 60
```

### Event Bindings

| Element | Event | Handler |
|---|---|---|
| `#character-svg` | `click` | `handleInteraction` |
| `#character-svg` | `keydown` (Enter/Space) | `handleInteraction` |
| `#cta-button` | `click` | `handleInteraction` |

---

## Data Models

### MOOD_LABELS (constant array)

```js
const MOOD_LABELS = [
  'CALM',
  'FOCUSED',
  'REFLECTIVE',
  'MYSTERIOUS',
  'PEACEFUL',
  'ENERGETIC'
];
```

Selection rule: uniform random, excluding the current mood (when `pool.length > 1`).

### QUOTES (constant array — exactly 8 items)

```js
const QUOTES = [
  "Stay calm. The night is still young. 🌙",
  "Sometimes silence says everything.",
  "Keep moving. Your light is still there. ✨",
  "Don't rush. Everything has its own time.",
  "Even the darkest night has a light.",
  "Take a breath. You're doing fine.",
  "Some nights are meant for reflection.",
  "Keep your focus. Keep your light."
];
```

Selection rule: uniform random, excluding the previously displayed quote (when `pool.length > 1`). `state.currentQuote` is `null` before the first interaction.

### Particle Configuration (per-particle, generated at runtime)

```js
{
  x: number,          // spawn x (character center, viewport-relative)
  y: number,          // spawn y (character center, viewport-relative)
  angle: number,      // 0–360°
  speed: number,      // 40–120 px/s (maps to translateY offset for keyframe)
  size: number,       // 4–10px (circle) or 8–14px (star ★)
  color: string,      // one of: '#6482FF', '#9664FF', '#E8E8FF'
  shape: 'circle' | 'star',
  opacity: number     // 0.6–1.0 initial
}
```

### SVG Character State Model

The character has two visual states, controlled by the presence/absence of `.character-active` on `#character-svg`:

| State | Class | `#char-eyes` opacity | `#char-accent` opacity | `#char-glow` filter |
|---|---|---|---|---|
| Neutral | (none) | 1.0 | 0.4 | blur(4px) |
| Active | `.character-active` | 1.0 → slight scale via CSS | 0.9 | blur(8px) |

Transitions between states use `transition: opacity 300ms ease, filter 300ms ease` on the relevant SVG groups.

### SVG Character Specification

Viewbox: `0 0 200 300`  
Color palette (≤ 5 fills):

| Token | Hex | Usage |
|---|---|---|
| `--char-coat` | `#0D0D1A` | Coat / body shape |
| `--char-navy` | `#1A1A3E` | Collar, lapels, trousers |
| `--char-skin` | `#C8A882` | Head / face |
| `--char-accent` | `#E8E8FF` | Hair highlight, moonlight reflection on lapel |
| `--char-glow` | `#7B6FC4` | Elliptical glow beneath feet |

SVG groups:

| Group ID | Description | Elements |
|---|---|---|
| `#char-glow` | Elliptical radial glow under feet | `<ellipse>` with radialGradient fill |
| `#char-body` | Coat torso with angular collar | `<path>` (2–3 shapes) |
| `#char-head` | Oval head | `<ellipse>` |
| `#char-hair` | Slicked-back dark hair | `<path>` |
| `#char-eyes` | Two almond-shaped eyes (neutral) | 2× `<ellipse>` |
| `#char-eyes-active` | Alternate eye wideness (opacity 0 by default) | 2× `<ellipse>` |
| `#char-accent` | Moonlight shoulder/lapel glint | `<path>` or `<ellipse>` |

Total distinct SVG shape elements: ≤ 18 (well within the ≤ 20 limit of Requirement 3.1).

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Counter Strict Monotonicity

*For any* sequence of N valid (non-debounced) interactions starting from `interactionCount = 0`, the counter value after all N interactions SHALL equal exactly N — it never decrements, never double-increments, and never skips.

**Validates: Requirements 6.3, 6.4, 6.5**

---

### Property 2: Quote Non-Repetition

*For any* sequence of two or more consecutive interactions on a pool of more than one quote, the quote selected at interaction `i` SHALL differ from the quote selected at interaction `i-1`.

**Validates: Requirements 8.4**

---

### Property 3: Mood Non-Repetition

*For any* sequence of two or more consecutive interactions on a mood set of more than one label, the mood selected at interaction `i` SHALL differ from the mood selected at interaction `i-1`.

**Validates: Requirements 7.4**

---

### Property 4: Particle DOM Cleanup

*For any* sequence of interactions, once 1300 ms have elapsed since the most recent interaction, the count of `.particle` elements in `#particle-container` SHALL equal zero (no leaked elements remain in the DOM).

**Validates: Requirements 12.3, 16.3**

---

### Property 5: Animation Debounce Exclusivity

*For any* interaction event that arrives while `state.isAnimating === true`, the `interactionCount` SHALL remain unchanged and no new particle elements SHALL be appended to `#particle-container`.

**Validates: Requirements 9.6, 10.4**

---

### Property 6: selectDifferent Correctness

*For any* non-empty pool array and any current value that exists in the pool, `selectDifferent(pool, current)` SHALL return a value that is a member of the pool AND, when `pool.length > 1`, the returned value SHALL NOT equal `current`.

**Validates: Requirements 7.4, 8.4** (this is the pure-function core relied upon by Properties 2 and 3)

---

## Error Handling

### CDN Failure (Tailwind / Google Fonts)

- **Tailwind CDN fails**: The inline `<style>` block includes critical layout rules (flex centering, fixed background, glassmorphism `.glass` class, all keyframe animations) as fallback. The app remains functional and visually coherent using browser default sans-serif.
- **Google Fonts fails**: The font stack falls back to `'Manrope', 'Inter', system-ui, sans-serif` declared on the `body` element. Text remains legible.
- Detection: no active detection required — CSS cascade handles fallback silently.

### SVG Render Failure

- An `<img>` fallback with equivalent dimensions is placed inside a `<noscript>`-style fallback or as a hidden sibling that becomes visible via CSS if the SVG element has zero rendered size.
- In practice, all modern browsers support inline SVG; the fallback handles edge cases.

### `backdrop-filter` Unsupported

- The `.glass` class includes a `background: rgba(255,255,255,0.12)` rule before the `backdrop-filter` declaration. Browsers that do not support `backdrop-filter` (legacy Edge, older Firefox) will render a semi-opaque container that preserves visual contrast against the dark background (Requirement 5.6).

### Character Element Not in DOM on Click

- The `handleInteraction()` function calls `document.getElementById('character-svg')` defensively before applying CSS classes. If the element is `null`, the function logs a warning and exits without throwing (Requirement 10.5).

### Particle Container Full (≥ 60 particles)

- Before spawning each particle, `spawnParticles()` checks `particleContainer.children.length`. If the count is ≥ 60, the spawn loop exits early. No new particles are added, existing in-flight particles are unaffected (Requirement 12.6).

### Interaction During Animation (Debounce)

- `state.isAnimating` is set to `true` at the start of `handleInteraction()` and `false` after `setTimeout(400ms)`. Any call that finds `isAnimating === true` returns immediately, ensuring no duplicate effects fire (Requirements 9.6, 10.4).

### Reduced Motion

- `@media (prefers-reduced-motion: reduce)` sets `animation-duration: 0.001ms !important` on all keyframe-animated elements and `transition-duration: 0.001ms !important` on transition-bearing elements. `spawnParticles()` also checks `window.matchMedia('(prefers-reduced-motion: reduce)').matches` and returns early without spawning particles (Requirement 16.5).

---

## Testing Strategy

### Dual Testing Approach

Light Night's logic is pure enough that its core functions — `selectDifferent`, counter increment, debounce guard, particle count enforcement — can be extracted and unit- or property-tested in isolation. The UI interactions (animations, DOM updates) are better covered by example-based browser tests.

### Unit Tests (Example-Based)

Focus areas:
- `selectDifferent(pool, current)` with specific inputs: single-item pool, current = first item, current = last item, current = null
- Counter starts at 0, increments correctly after one call, increments correctly after ten calls
- `spawnParticles` creates exactly `count` elements when container is empty
- `spawnParticles` does not exceed 60 total elements when called repeatedly
- Debounce: second call to `handleInteraction()` while `isAnimating = true` leaves counter unchanged

### Property-Based Tests (using fast-check via CDN in test harness)

Property-based testing is appropriate here because the app's two core selection functions (`selectDifferent` for moods and quotes) involve **universal correctness guarantees over all valid inputs** — not just specific examples. The input space (any pool, any current) is easily generated.

**Library**: [fast-check](https://github.com/dubzzz/fast-check) loaded via CDN in a separate `test.html` harness file. Each test runs a minimum of **100 iterations**.

**Tag format**: `// Feature: light-night, Property {N}: {property_text}`

#### P1 — Counter Strict Monotonicity
```
// Feature: light-night, Property 1: counter equals exact interaction count
fc.assert(fc.property(fc.integer({min: 1, max: 500}), (n) => {
  resetState();
  for (let i = 0; i < n; i++) triggerInteractionDirect();
  return state.interactionCount === n;
}), { numRuns: 100 });
```

#### P2 — Quote Non-Repetition
```
// Feature: light-night, Property 2: consecutive quotes differ
fc.assert(fc.property(fc.integer({min: 2, max: 200}), (n) => {
  resetState();
  let prev = null;
  for (let i = 0; i < n; i++) {
    const next = selectDifferent(QUOTES, prev);
    if (prev !== null && next === prev) return false;
    prev = next;
  }
  return true;
}), { numRuns: 100 });
```

#### P3 — Mood Non-Repetition
```
// Feature: light-night, Property 3: consecutive moods differ
fc.assert(fc.property(fc.integer({min: 2, max: 200}), (n) => {
  let prev = MOOD_LABELS[0];
  for (let i = 0; i < n; i++) {
    const next = selectDifferent(MOOD_LABELS, prev);
    if (next === prev) return false;
    prev = next;
  }
  return true;
}), { numRuns: 100 });
```

#### P4 — Particle DOM Cleanup
```
// Feature: light-night, Property 4: no particles remain after 1300ms
fc.assert(fc.asyncProperty(fc.integer({min: 1, max: 5}), async (interactions) => {
  for (let i = 0; i < interactions; i++) spawnParticles(randomCount());
  await sleep(1300);
  return particleContainer.children.length === 0;
}), { numRuns: 100 });
```

#### P5 — Animation Debounce Exclusivity
```
// Feature: light-night, Property 5: debounced calls do not increment counter
fc.assert(fc.property(fc.integer({min: 1, max: 50}), (extraCalls) => {
  resetState();
  state.isAnimating = true;
  for (let i = 0; i < extraCalls; i++) handleInteraction();
  return state.interactionCount === 0;
}), { numRuns: 100 });
```

#### P6 — selectDifferent Correctness
```
// Feature: light-night, Property 6: selectDifferent returns pool member, never repeats
fc.assert(fc.property(
  fc.array(fc.string(), {minLength: 2, maxLength: 20}),
  fc.nat(),
  (pool, idx) => {
    const current = pool[idx % pool.length];
    const result = selectDifferent(pool, current);
    return pool.includes(result) && result !== current;
  }
), { numRuns: 100 });
```

### Integration Tests (Browser)

Run in a real browser environment (e.g., Playwright or manual verification):
- Page loads and all UI elements visible at 320px, 768px, 1280px viewports
- Click/tap on character triggers all four visible effects simultaneously
- Keyboard Tab reaches character and CTA button; Enter triggers interaction
- `aria-live` regions announce updates (manual screen reader check)
- At 60+ rapid clicks, particle count does not exceed 60
- With `prefers-reduced-motion: reduce` emulated, no keyframe animations play

### Accessibility Verification

Full WCAG 2.1 AA validation requires manual testing with assistive technologies (NVDA, VoiceOver) in addition to automated checks. Automated tools (axe-core) can verify contrast ratios, ARIA attributes, and keyboard focus order.
