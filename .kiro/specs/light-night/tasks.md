# Implementation Plan: Light Night

## Overview

Implement Light Night as a single self-contained `light-night.html` file with no build step, no backend, and no external JS/CSS beyond a Tailwind CDN script tag and a Google Fonts link. The file is split into three logical sections: a `<head>` block (CDN tags + inline `<style>`), a `<body>` block (markup + inline SVG character + all glassmorphism UI), and a `<script>` block (vanilla ES6 state machine, event handlers, animation triggers, and particle system). A separate `test.html` harness validates the six correctness properties using fast-check.

---

## Tasks

- [x] 1. Create HTML scaffold and head section
  - Create `light-night.html` with `<!DOCTYPE html>`, `lang="en"`, and a `<meta name="viewport" content="width=device-width, initial-scale=1.0">` tag
  - Add Tailwind CSS via the official CDN `<script src="https://cdn.tailwindcss.com">` tag in `<head>`
  - Add Google Font (Manrope) via `<link rel="preconnect">` + `<link rel="stylesheet">` in `<head>`
  - Add an inline `<style>` block containing:
    - CSS custom properties (`--char-coat`, `--char-navy`, `--char-skin`, `--char-accent`, `--char-glow`)
    - `.glass` utility class: `bg-white/10 backdrop-blur-xl rounded-3xl border border-white/20` with `rgba(255,255,255,0.12)` pre-declaration for `backdrop-filter` fallback
    - `@keyframes float`, `@keyframes scale-bounce`, `@keyframes pulse-glow`, `@keyframes particle-fly`, `@keyframes fade-in`
    - `@media (prefers-reduced-motion: reduce)` block setting `animation-duration: 0.001ms !important` and `transition-duration: 0.001ms !important` on all animated elements
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.6, 16.5_

- [x] 2. Implement background and base layout
  - [x] 2.1 Implement fixed full-viewport dark gradient background
    - Apply CSS gradient on `<body>` progressing through `#0F1020 → #17152D → #242044 → #34305C`
    - Set `position: fixed`, `width: 100vw`, `height: 100vh`, `background-attachment: fixed` so content scrolls over it
    - Add centered radial gradient overlay `<div>` sized ≤60% viewport with opacity between 0.05 and 0.20
    - Add three ambient glow `<div>` accents using blue (`#6482FF`), violet (`#9664FF`), white (`#E8E8FF`) with blur 40–120px and opacity 0.05–0.20, constrained to the central 50% of viewport
    - _Requirements: 2.1, 2.2, 2.3, 2.4_

  - [x] 2.2 Build `#app` flex column layout
    - Create `<div id="app">` with `flex flex-col min-h-screen items-center justify-center gap-6 relative z-10`
    - Ensure no horizontal scrollbar appears at any viewport width from 320px to 2560px
    - _Requirements: 14.1, 14.4_

- [x] 3. Implement typography heading and subtitle
  - [x] 3.1 Add "LIGHT NIGHT" heading and subtitle
    - Create `<header>` containing an `<h1>` with text `LIGHT NIGHT` in uppercase, `letter-spacing: 0.15em`, using the Manrope font stack with `system-ui, sans-serif` fallback
    - Add subtitle `<p>` with text "Find your light in the quiet of the night."
    - Apply responsive font sizes: `text-3xl sm:text-5xl md:text-7xl lg:text-8xl` (maps to ~28–120px range across breakpoints)
    - Verify heading color achieves ≥4.5:1 contrast ratio against the dark background (use `#F0F0FF` or equivalent)
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6_

- [x] 4. Build stats bar with Counter and Mood components
  - [x] 4.1 Implement `#counter-component` glassmorphism container
    - Create `<div id="stats-bar">` as `flex flex-row gap-4`
    - Inside it, add `<div id="counter-component">` with `.glass` class and inner markup: `<span>INTERACTIONS</span>` label and `<span id="counter-value" aria-live="polite" aria-atomic="true">0</span>`
    - Initialize displayed value to `0`
    - _Requirements: 5.1, 6.1, 6.2, 15.5_

  - [x] 4.2 Implement `#status-display` glassmorphism container
    - Add `<div id="status-display">` with `.glass` class, inner markup: `<span>MOOD</span>` label and `<span id="mood-label">CALM</span>`
    - Apply `transition: opacity 300ms ease` on `#mood-label` for text-change animation
    - _Requirements: 5.2, 7.1, 7.2, 7.5_

- [x] 5. Implement inline SVG character
  - [x] 5.1 Create SVG character with all 7 groups
    - Create `<svg id="character-svg" viewBox="0 0 200 300" ...>` with `role="button"`, `tabindex="0"`, `aria-label="Interact with Light Night"`
    - Implement `<g id="char-glow">` with `<ellipse>` filled by a `<radialGradient>` using `--char-glow` (`#7B6FC4`) color, centered below the feet, radius ≤50% of character width on each side
    - Implement `<g id="char-body">` with 2–3 `<path>` elements for coat/torso using `--char-coat` (`#0D0D1A`) and `--char-navy` (`#1A1A3E`)
    - Implement `<g id="char-head">` with `<ellipse>` filled with `--char-skin` (`#C8A882`)
    - Implement `<g id="char-hair">` with `<path>` for slicked-back dark hair using `--char-coat`
    - Implement `<g id="char-eyes">` with 2× `<ellipse>` for almond-shaped neutral eyes
    - Implement `<g id="char-eyes-active">` with alternate eye ellipses set to `opacity: 0` by default
    - Implement `<g id="char-accent">` with `<path>` or `<ellipse>` for shoulder/lapel moonlight glint using `--char-accent` (`#E8E8FF`)
    - Ensure total distinct SVG shape elements ≤18 and distinct fill colors ≤5
    - _Requirements: 3.1, 3.2, 3.6, 13.1_

  - [x] 5.2 Wrap character in container with glow ring and sizing
    - Create `<div id="character-container">` with `.glass` class, `relative`, and the `float` keyframe animation
    - Add `<div id="glow-ring">` with `absolute` positioning, centered behind the SVG (lower z-index), and initial `opacity: 0`
    - Apply `aspect-square w-60 sm:w-72 md:w-80 lg:w-96 xl:w-[640px] max-w-[640px]` to the SVG so min ≥240px and max ≤640px
    - Horizontally and vertically center the character within the viewport using Tailwind centering utilities
    - Add `-webkit-tap-highlight-color: transparent` on `#character-svg`
    - _Requirements: 3.3, 3.4, 3.5, 3.7, 14.2, 14.6_

  - [x] 5.3 Add keyboard focus indicator on SVG character
    - Apply `focus:outline focus:outline-2 focus:outline-offset-4 focus:outline-blue-400` or equivalent inline style to ensure a visible 2px focus ring
    - _Requirements: 15.1_

- [x] 6. Implement reaction area and CTA button
  - [x] 6.1 Build `#reaction-area` glassmorphism container
    - Create `<div id="reaction-area" aria-live="polite" aria-atomic="false">` with `.glass` class
    - Leave inner content empty on load (no placeholder text)
    - Add a fade-in CSS animation class (200–600ms) that is applied each time new quote text is injected
    - _Requirements: 5.3, 8.2, 8.5, 15.4_

  - [x] 6.2 Build `#cta-button` as native `<button>`
    - Create `<button id="cta-button">` with label text `Touch the Light` chosen at initialization and not changing during the session
    - Apply `.glass` class plus a soft glow via `box-shadow: 0 0 12px rgba(100,130,255,0.35)`
    - Add `hover:` style applying expanded glow ≤16px CSS pixels with `transition-duration: 150ms`
    - Ensure the button's interactive hit area is ≥44×44px via explicit `min-w-[44px] min-h-[44px] px-6 py-3`
    - Add `-webkit-tap-highlight-color: transparent`
    - _Requirements: 5.4, 9.1, 9.3, 9.4, 9.5, 14.6_

- [x] 7. Add particle container
  - Create `<div id="particle-container">` placed as a direct child of `<body>`, outside `#app`, with `position: fixed; inset: 0; pointer-events: none; z-index: 50;`
  - _Requirements: 12.1, 12.4_

- [x] 8. Implement JavaScript state and data layer
  - [x] 8.1 Define state object and constant data arrays
    - Write `const state = { interactionCount: 0, currentMood: 'CALM', currentQuote: null, isAnimating: false }` in the `<script>` block
    - Write `const MOOD_LABELS = ['CALM', 'FOCUSED', 'REFLECTIVE', 'MYSTERIOUS', 'PEACEFUL', 'ENERGETIC']` (6 items)
    - Write `const QUOTES = [...]` with exactly the 8 specified quote strings
    - _Requirements: 7.1, 7.2, 8.1, 6.2_

  - [x] 8.2 Implement `selectDifferent(pool, current)` pure helper
    - Write the function: given a non-empty array `pool` and a `current` value, return a uniformly random element from `pool` that is not equal to `current` (when `pool.length > 1`)
    - When `pool.length === 1`, return `pool[0]` regardless of `current`
    - Export/expose the function on `window` so it is accessible from the test harness
    - _Requirements: 7.3, 7.4, 8.3, 8.4_

  - [ ]* 8.3 Write property test for `selectDifferent` correctness (Property 6)
    - **Property 6: selectDifferent returns a pool member and never equals current when pool.length > 1**
    - **Validates: Requirements 7.4, 8.4**

- [x] 9. Implement `handleInteraction()` core handler
  - [x] 9.1 Write `handleInteraction()` with debounce guard and state mutations
    - At function entry: `if (state.isAnimating) return;` then `state.isAnimating = true`
    - Increment `state.interactionCount` by 1
    - Call `selectDifferent(MOOD_LABELS, state.currentMood)` and assign to `state.currentMood`
    - Call `selectDifferent(QUOTES, state.currentQuote)` and assign to `state.currentQuote`
    - Update DOM: `#counter-value` innerText, `#mood-label` innerText (with opacity transition), `#reaction-area` innerText with fade-in class re-trigger
    - All DOM updates must complete within 100ms of the event
    - Set `setTimeout(() => { state.isAnimating = false; }, 400)` to release the lock after the scale animation cycle
    - Expose `handleInteraction` and `state` on `window` for test harness access
    - _Requirements: 6.3, 6.4, 7.3, 7.4, 7.5, 8.3, 8.4, 9.2, 9.6_

  - [ ]* 9.2 Write property test for Counter Strict Monotonicity (Property 1)
    - **Property 1: After N non-debounced interactions from count=0, interactionCount === N**
    - **Validates: Requirements 6.3, 6.4, 6.5**

  - [ ]* 9.3 Write property test for Animation Debounce Exclusivity (Property 5)
    - **Property 5: Calls to handleInteraction while isAnimating===true leave interactionCount unchanged**
    - **Validates: Requirements 9.6, 10.4**

- [x] 10. Implement CSS animation triggers
  - [x] 10.1 Implement `triggerScaleAnimation()` on `#character-svg`
    - Add `.scale-bounce` class to the character SVG element; remove it after 400ms via `setTimeout`
    - If called while `.scale-bounce` is already applied, remove and force a reflow (`element.offsetWidth`) before re-adding to restart the animation from the beginning
    - The CSS `@keyframes scale-bounce` rule must scale from `1.0 → 1.25 → 1.0` within 400ms using `cubic-bezier(0.36, 0.07, 0.19, 0.97)`
    - _Requirements: 10.1, 10.2, 10.3, 10.4_

  - [x] 10.2 Implement `triggerGlowAnimation()` on `#glow-ring`
    - Add `.pulse-glow` class to `#glow-ring`; remove after 600ms via `setTimeout`
    - Cancel and restart on rapid re-trigger (same reflow trick as scale animation)
    - The CSS `@keyframes pulse-glow` must expand a radial glow from opacity 0 to peak (0.1–0.6) and back to 0, capping expansion at 150% of the character bounding-box diameter, all within 600ms
    - Ensure `#glow-ring` z-index is lower than `#character-svg` so glow renders behind the character
    - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5_

  - [x] 10.3 Implement `triggerCharacterActive()` for SVG expression change
    - Add `.character-active` CSS class to `#character-svg` within 16ms of the interaction (call directly in `handleInteraction`, not in a setTimeout)
    - The CSS rule for `.character-active #char-eyes-active` should set `opacity: 1`, and `.character-active #char-accent` should set `opacity: 0.9`, and `.character-active #char-glow` should apply `filter: blur(8px)`
    - Apply `transition: opacity 300ms ease, filter 300ms ease` on `#char-eyes-active`, `#char-accent`, and `#char-glow`
    - Call `setTimeout(() => character.classList.remove('character-active'), 500)` after triggering; reset the timer if called again while the previous timer is pending
    - Null-check `document.getElementById('character-svg')` before applying class; log a warning and return early if null
    - _Requirements: 3.6, 13.1, 13.2, 13.3, 13.4, 13.5, 10.5_

- [x] 11. Checkpoint — core interaction loop complete
  - Ensure all tests pass, ask the user if questions arise.

- [x] 12. Implement particle system
  - [x] 12.1 Implement `spawnParticles(count)` with DOM creation and cleanup
    - At function entry, check `window.matchMedia('(prefers-reduced-motion: reduce)').matches` — return early without spawning if true
    - Check `particleContainer.children.length`; if ≥60, exit early without spawning any new particles
    - For each of `count` particles (6–12):
      - Get character center via `characterSvg.getBoundingClientRect()`
      - Randomize: `angle` 0–360°, `speed` 40–120 px/s, `size` 4–10px (circle) or 8–14px (star `★`), `color` from `['#6482FF', '#9664FF', '#E8E8FF']`, `opacity` 0.6–1.0, `shape` circle or star
      - Create `<div class="particle">` with `position: fixed`, spawn position at character center, apply CSS custom properties `--angle`, `--speed`, `--size`, `--color`, `--opacity` for use in `@keyframes particle-fly`
      - Append to `#particle-container`
      - Schedule removal: `setTimeout(() => el.remove(), 1300)` (1200ms animation + 100ms buffer)
    - Expose `spawnParticles` on `window` for test harness
    - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.5, 12.6, 16.3, 16.5_

  - [ ]* 12.2 Write property test for Particle DOM Cleanup (Property 4)
    - **Property 4: After 1300ms from the last spawned batch, #particle-container has zero .particle children**
    - **Validates: Requirements 12.3, 16.3**

- [x] 13. Wire up all event bindings
  - [x] 13.1 Bind all three interaction entry points
    - `document.getElementById('character-svg').addEventListener('click', handleInteraction)`
    - `document.getElementById('character-svg').addEventListener('keydown', e => { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); handleInteraction(); } })`
    - `document.getElementById('cta-button').addEventListener('click', handleInteraction)`
    - Call `triggerScaleAnimation()`, `triggerGlowAnimation()`, `triggerCharacterActive()`, and `spawnParticles(randomBetween(6, 12))` inside `handleInteraction()` after the debounce guard
    - _Requirements: 9.2, 10.1, 11.1, 12.1, 15.1, 15.2_

- [x] 14. Responsive and accessibility polish
  - [x] 14.1 Verify and fix responsive layout at all breakpoints
    - Open `light-night.html` in a browser and resize to 320px, 375px, 768px, 1024px, 1280px, 1920px, and 2560px widths
    - Confirm no horizontal scrollbar, no clipped content, character centered within 2px, all text ≥12px, heading ≥28px on mobile and ≥48px on desktop
    - Confirm all interactive tap targets ≥44×44px (character, CTA button)
    - _Requirements: 14.1, 14.2, 14.3, 14.4, 14.5, 14.7_

  - [x] 14.2 Verify accessibility attributes and keyboard navigation
    - Confirm `#reaction-area` has `aria-live="polite"` and `aria-atomic="false"`
    - Confirm `#counter-value` has `aria-live="polite"`
    - Confirm `#character-svg` has `role="button"`, `tabindex="0"`, `aria-label="Interact with Light Night"`, and a visible 2px focus outline
    - Confirm `#cta-button` is a native `<button>` reachable via Tab
    - Confirm `prefers-reduced-motion` disables all animations and particle spawning
    - _Requirements: 15.1, 15.2, 15.3, 15.4, 15.5, 15.6, 16.5_

- [x] 15. Create property-based test harness (test.html)
  - [x] 15.1 Set up `test.html` with fast-check via CDN
    - Create `test.html` in the same directory as `light-night.html`
    - Load fast-check via CDN: `<script src="https://cdn.jsdelivr.net/npm/fast-check@3/lib/bundle/fast-check.min.js"></script>`
    - Load `light-night.html` logic in an isolated scope (inline script that imports only the testable functions via a `<script type="module">` referencing the functions exported on `window`)
    - Add a simple pass/fail reporter to `<div id="results">`
    - _Requirements: (test infrastructure)_

  - [x] 15.2 Implement P1 — Counter Strict Monotonicity test
    - Use `fc.assert(fc.property(fc.integer({min:1, max:500}), n => { resetState(); for(let i=0;i<n;i++) triggerInteractionDirect(); return state.interactionCount===n; }), {numRuns:100})`
    - Tag: `// Feature: light-night, Property 1: counter equals exact interaction count`
    - **Validates: Requirements 6.3, 6.4, 6.5**

  - [x] 15.3 Implement P2 — Quote Non-Repetition test
    - Use `fc.assert(fc.property(fc.integer({min:2,max:200}), n => { let prev=null; for(let i=0;i<n;i++){ const next=selectDifferent(QUOTES,prev); if(prev!==null && next===prev) return false; prev=next; } return true; }), {numRuns:100})`
    - Tag: `// Feature: light-night, Property 2: consecutive quotes differ`
    - **Validates: Requirements 8.4**

  - [x] 15.4 Implement P3 — Mood Non-Repetition test
    - Use `fc.assert(fc.property(fc.integer({min:2,max:200}), n => { let prev=MOOD_LABELS[0]; for(let i=0;i<n;i++){ const next=selectDifferent(MOOD_LABELS,prev); if(next===prev) return false; prev=next; } return true; }), {numRuns:100})`
    - Tag: `// Feature: light-night, Property 3: consecutive moods differ`
    - **Validates: Requirements 7.4**

  - [ ]* 15.5 Implement P4 — Particle DOM Cleanup test
    - Use async property test with 1300ms sleep; verify `particleContainer.children.length === 0` after wait
    - Tag: `// Feature: light-night, Property 4: no particles remain after 1300ms`
    - **Validates: Requirements 12.3, 16.3**

  - [x] 15.6 Implement P5 — Animation Debounce Exclusivity test
    - Use `fc.assert(fc.property(fc.integer({min:1,max:50}), n => { resetState(); state.isAnimating=true; for(let i=0;i<n;i++) handleInteraction(); return state.interactionCount===0; }), {numRuns:100})`
    - Tag: `// Feature: light-night, Property 5: debounced calls do not increment counter`
    - **Validates: Requirements 9.6, 10.4**

  - [x] 15.7 Implement P6 — selectDifferent Correctness test
    - Use `fc.assert(fc.property(fc.array(fc.string(),{minLength:2,maxLength:20}), fc.nat(), (pool,idx) => { const cur=pool[idx%pool.length]; const r=selectDifferent(pool,cur); return pool.includes(r) && r!==cur; }), {numRuns:100})`
    - Tag: `// Feature: light-night, Property 6: selectDifferent returns pool member, never repeats`
    - **Validates: Requirements 7.4, 8.4**

- [x] 16. Final checkpoint — all tests and visual checks pass
  - Open `test.html` in a browser and confirm all 6 property tests pass (P1–P6)
  - Open `light-night.html` and verify the complete interaction sequence: click character → counter increments, mood updates, quote fades in, character bounces, glow ring pulses, particles burst and clean up
  - Ensure all tests pass, ask the user if questions arise.

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP delivery
- Each task references specific requirements for full traceability back to the spec
- The entire deliverable is two files: `light-night.html` (production) and `test.html` (test harness)
- Property tests run in a browser context because the app uses DOM APIs — no Node.js test runner required
- All animations must use only `transform` and `opacity` to avoid layout reflow (Requirement 16.2)
- The `selectDifferent` function and all stateful objects must be exposed on `window` for the test harness to import without a build step

---

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["2.1", "2.2"] },
    { "id": 1, "tasks": ["3.1", "4.1", "4.2", "7"] },
    { "id": 2, "tasks": ["5.1", "6.1", "6.2", "8.1"] },
    { "id": 3, "tasks": ["5.2", "8.2"] },
    { "id": 4, "tasks": ["5.3", "8.3", "9.1"] },
    { "id": 5, "tasks": ["9.2", "9.3", "10.1", "10.2", "10.3"] },
    { "id": 6, "tasks": ["12.1", "13.1"] },
    { "id": 7, "tasks": ["12.2", "14.1", "14.2"] },
    { "id": 8, "tasks": ["15.1"] },
    { "id": 9, "tasks": ["15.2", "15.3", "15.4", "15.6", "15.7"] },
    { "id": 10, "tasks": ["15.5"] }
  ]
}
```
