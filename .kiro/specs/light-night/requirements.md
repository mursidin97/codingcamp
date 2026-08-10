# Requirements Document

## Introduction

Light Night is a single-page interactive web application delivered as a single HTML file with no build step or backend. It features a minimalist, masculine character called "Light Night" centered on a dark night-themed background. The user interacts with the character by clicking or tapping, triggering random reaction quotes, subtle mood changes, visual particle effects, and micro-animations. The application is designed to feel premium, calm, and immediately engaging across all modern devices and screen sizes.

## Glossary

- **App**: The Light Night single-page interactive web application.
- **Character**: The inline SVG illustration of the Light Night figure displayed at the center of the App.
- **Interaction**: A single user click or tap event registered on the Character or the CTA Button.
- **Reaction_Quote**: One of the predefined text strings displayed below the Character after an Interaction.
- **Mood_Label**: A single-word status string (CALM, FOCUSED, REFLECTIVE, MYSTERIOUS, PEACEFUL, ENERGETIC) displayed in the Status Display component.
- **Interaction_Counter**: The numeric count of total Interactions in the current session, displayed in the Counter Component.
- **Counter_Component**: The glassmorphism UI element that shows the current Interaction_Counter value.
- **Status_Display**: The glassmorphism UI element that shows the current Mood_Label.
- **Reaction_Area**: The text area below the Character that displays the current Reaction_Quote.
- **CTA_Button**: The primary call-to-action button labeled "Touch the Light" or "Enter the Night".
- **Particle_Effect**: The floating visual elements (glowing dots, stars, light waves) spawned on each Interaction.
- **Scale_Animation**: The brief grow-then-return CSS transform applied to the Character on each Interaction.
- **Glow_Animation**: The soft radial light pulse emitted from the Character on each Interaction.
- **Background**: The full-viewport dark gradient canvas behind all other elements.
- **Glassmorphism_Container**: A UI container styled with translucent background, backdrop blur, rounded corners, and a subtle border.

---

## Requirements

### Requirement 1: Single-File Delivery

**User Story:** As a user, I want the app to load instantly from a single HTML file, so that I can use it without any installation, build tools, or server.

#### Acceptance Criteria

1. THE App SHALL be delivered as a single self-contained HTML file with no external JavaScript or CSS files beyond CDN links.
2. THE App SHALL load Tailwind CSS exclusively via the official Tailwind CDN `<script>` tag.
3. THE App SHALL load exactly one Google Font selected from Inter, Manrope, or Plus Jakarta Sans via a `<link>` tag placed in the document `<head>` section.
4. THE App SHALL contain no server-side code, no build configuration files, and no framework dependencies.
5. WHEN the HTML file is opened directly in a browser that supports ES6 and CSS3 without a server, THE App SHALL render all visible UI elements and respond to user interactions without errors.
6. IF the CDN resources fail to load, THEN THE App SHALL remain functional using inline fallback styles and the browser's default sans-serif font, with no loss of core functionality.

---

### Requirement 2: Background and Atmosphere

**User Story:** As a user, I want to be immersed in a premium night-themed atmosphere, so that the visual experience feels calm, elegant, and modern.

#### Acceptance Criteria

1. THE Background SHALL cover the full viewport width and height at all screen sizes using a dark gradient that progresses through `#0F1020`, `#17152D`, `#242044`, and `#34305C`.
2. THE Background SHALL display a centered radial gradient overlay no larger than 60% of the viewport width and height to create a soft glowing effect behind the Character.
3. WHILE the App is displayed, THE Background SHALL remain fixed and non-scrolling, and any page content that exceeds the viewport height SHALL scroll over the fixed Background.
4. THE App SHALL apply subtle blue, violet, and white ambient glow accents with opacity between 0.05 and 0.20 and blur radius between 40px and 120px, constrained to the central 50% of the viewport width aligned to the Character region.

---

### Requirement 3: Character Rendering

**User Story:** As a user, I want to see a stylish, large, minimalist male character at the center of the screen, so that the character immediately becomes the visual focal point.

#### Acceptance Criteria

1. THE App SHALL render the Character as an inline SVG element using a clean, minimalist vector illustration style with no more than 20 distinct SVG shape elements and no raster image references.
2. THE Character SHALL have a masculine, elegant, calm, and mysterious visual identity — not childish or cartoonish — achieved through sharp or neutral facial features, a closed or subtly downward mouth, and muted or dark color tones containing no more than 5 distinct fill colors.
3. THE Character SHALL be centered horizontally and vertically within the viewport using CSS, such that the Character's bounding box midpoint is within 2px of the viewport center on any viewport width between 320px and 2560px.
4. THE Character SHALL appear to stand or slightly float above a soft glowing elliptical surface rendered directly below the Character's feet, where the glow radius extends no more than 50% of the Character's rendered width on each side.
5. THE App SHALL size the Character using Tailwind's `aspect-square` utility so the Character maintains a 1:1 aspect ratio, with a minimum rendered size of 240px and a maximum rendered size of 640px, at all viewport sizes.
6. THE Character SHALL include at least one SVG group element with a designated CSS class such that applying a predefined alternate CSS class to that group produces a visually distinct change — a difference of at least 5 degrees in a path angle, or at least 4px displacement of an element — representing a subtle expression or pose variation on Interaction.
7. IF the inline SVG element fails to render or is unsupported by the browser, THEN THE App SHALL display a fallback static image of the Character with equivalent dimensions and centering.

---

### Requirement 4: Typography and Heading

**User Story:** As a user, I want to see clear, premium typography identifying the experience, so that the brand and subtitle are immediately legible.

#### Acceptance Criteria

1. THE App SHALL display the heading text "LIGHT NIGHT" in uppercase letters with a letter-spacing of at least 0.1em using the chosen Google Font.
2. THE App SHALL display the subtitle "Find your light in the quiet of the night." positioned below the heading.
3. THE App SHALL render heading text in a color with a contrast ratio of at least 4.5:1 against the dark background, as measured by the WCAG 2.1 contrast algorithm.
4. WHEN the viewport width is between 320px and 599px, THE App SHALL render the heading at a font size between 28px and 48px.
5. WHEN the viewport width is between 600px and 2560px, THE App SHALL render the heading at a font size between 48px and 120px.
6. IF the chosen Google Font fails to load, THE App SHALL render the heading and subtitle using a serif or sans-serif system font fallback so that the text remains visible and legible.

---

### Requirement 5: Glassmorphism UI Containers

**User Story:** As a user, I want UI elements to feel translucent and premium, so that they complement the night-themed atmosphere without distracting from the Character.

#### Acceptance Criteria

1. THE Counter_Component SHALL be styled as a Glassmorphism_Container using `bg-white/10`, `backdrop-blur-xl`, `rounded-3xl`, and `border border-white/20` Tailwind classes.
2. THE Status_Display SHALL be styled as a Glassmorphism_Container using the same class set as the Counter_Component.
3. THE Reaction_Area SHALL be styled as a Glassmorphism_Container using `bg-white/10`, `backdrop-blur-xl`, `rounded-3xl`, and `border border-white/20` Tailwind classes.
4. THE CTA_Button SHALL be styled as a Glassmorphism_Container with `bg-white/10`, `backdrop-blur-xl`, `rounded-3xl`, and `border border-white/20` Tailwind classes, plus an additional soft glow effect applied via a CSS box-shadow with a spread radius no greater than 20px or a Tailwind ring utility of ring width between 1px and 4px.
5. WHILE the App is displayed, THE Glassmorphism_Container elements SHALL maintain their translucency against the Background without rendering as fully opaque, meaning the background-color opacity SHALL be no greater than 50%.
6. IF the user's browser does not support the CSS `backdrop-filter` property, THEN THE Glassmorphism_Container SHALL fall back to a background-color opacity between 30% and 60% to preserve visual contrast against the Background.

---

### Requirement 6: Interaction Counter

**User Story:** As a user, I want to see how many times I have interacted with the character, so that I feel a sense of progression and engagement.

#### Acceptance Criteria

1. THE Counter_Component SHALL display the label "INTERACTIONS" and the current Interaction_Counter value.
2. THE Interaction_Counter SHALL be initialized to `0` when the App first loads.
3. WHEN an Interaction occurs, THE Counter_Component SHALL increment the Interaction_Counter by exactly 1.
4. THE Counter_Component SHALL update the displayed number within 100 milliseconds after each Interaction.
5. THE Interaction_Counter SHALL persist its value for the entire browser session and SHALL reset to `0` only on full page reload.
6. THE Interaction_Counter SHALL support values in the range of 0 to 999,999 without display truncation or overflow.
7. IF an Interaction event is received while the Counter_Component is not mounted, THEN THE System SHALL discard the increment and preserve the last valid Interaction_Counter value.

---

### Requirement 7: Mood / Status Display

**User Story:** As a user, I want to see the character's mood change with each interaction, so that the app feels alive and dynamic.

#### Acceptance Criteria

1. THE Status_Display SHALL show one Mood_Label selected from the set: CALM, FOCUSED, REFLECTIVE, MYSTERIOUS, PEACEFUL, ENERGETIC.
2. THE App SHALL initialize the Status_Display with the Mood_Label "CALM" on page load.
3. WHEN an Interaction occurs, THE App SHALL randomly select a Mood_Label from the full set with equal probability for each label and update the Status_Display within 100ms of the Interaction.
4. WHEN an Interaction occurs, THE App SHALL ensure the newly selected Mood_Label is different from the previously displayed Mood_Label, provided the set contains more than one option.
5. THE Status_Display SHALL apply a CSS transition of between 200ms and 500ms duration when the Mood_Label text changes.
6. IF the Status_Display fails to update within 500ms of an Interaction, THEN THE App SHALL retain the previously displayed Mood_Label unchanged.

---

### Requirement 8: Reaction Quotes

**User Story:** As a user, I want to see a different inspirational quote from the character with each click, so that each interaction feels fresh and meaningful.

#### Acceptance Criteria

1. THE App SHALL maintain a pool of exactly the following eight Reaction_Quotes:
   - "Stay calm. The night is still young. 🌙"
   - "Sometimes silence says everything."
   - "Keep moving. Your light is still there. ✨"
   - "Don't rush. Everything has its own time."
   - "Even the darkest night has a light."
   - "Take a breath. You're doing fine."
   - "Some nights are meant for reflection."
   - "Keep your focus. Keep your light."
2. THE Reaction_Area SHALL display no text and no placeholder prompt on page load before any Interaction has occurred.
3. WHEN an Interaction occurs, THE App SHALL randomly select one Reaction_Quote from the pool with uniform probability and display it in the Reaction_Area.
4. WHEN an Interaction occurs, THE App SHALL ensure the newly selected Reaction_Quote is different from the previously displayed Reaction_Quote, provided the pool contains more than one quote.
5. WHEN a Reaction_Quote is displayed, THE App SHALL animate its appearance in the Reaction_Area using a fade-in CSS transition with a duration between 200ms and 600ms.
6. THE App SHALL allow the addition of Reaction_Quotes to the pool without requiring changes to any code other than the quote pool array.

---

### Requirement 9: CTA Button

**User Story:** As a user, I want a clearly labeled button to initiate my first interaction, so that the app immediately communicates how to engage with it.

#### Acceptance Criteria

1. THE CTA_Button SHALL display a fixed label of either "Touch the Light" or "Enter the Night", chosen at application initialization and remaining unchanged for the duration of the session.
2. WHEN the CTA_Button is clicked or tapped, THE App SHALL initiate the Interaction sequence within 300ms of the input event.
3. THE CTA_Button SHALL be visible and not clipped, hidden by overflow, or overlapped by other elements at all supported viewport sizes, and SHALL be reachable via keyboard Tab navigation.
4. WHEN a pointer device cursor enters the CTA_Button's hover region, THE App SHALL apply a soft glow effect with shadow or filter expansion no greater than 16 CSS pixels and a CSS transition duration no greater than 200ms.
5. THE CTA_Button's interactive hit area SHALL be at least 44×44 CSS pixels on all devices.
6. IF the CTA_Button is activated while an Interaction sequence is already in progress, THEN THE App SHALL ignore the activation event and not trigger a duplicate Interaction sequence.

---

### Requirement 10: Scale Animation on Interaction

**User Story:** As a user, I want the character to react visually when I click or tap, so that the interaction feels immediate and satisfying.

#### Acceptance Criteria

1. WHEN a click or tap event occurs on the Character, THE Character SHALL scale up to between 1.1x and 1.5x its original size using a CSS `transform: scale()` and then return to its original size (scale 1.0).
2. THE Scale_Animation SHALL complete its full grow-and-return cycle within 400ms of the triggering event.
3. THE Scale_Animation SHALL use a CSS `transition` or `animation` property rather than JavaScript-driven frame-by-frame updates.
4. WHEN a click or tap event occurs on the Character while a Scale_Animation is already in progress, THE App SHALL cancel the current Scale_Animation and restart it from the initial scale value rather than stacking or queuing animations.
5. IF the Character element is not present in the DOM when a click or tap event occurs, THEN THE App SHALL ignore the event without throwing an error or entering an inconsistent state.

---

### Requirement 11: Glow Animation on Interaction

**User Story:** As a user, I want a soft light pulse to emanate from the character on each click, so that the app reinforces the night-light theme through motion.

#### Acceptance Criteria

1. WHEN a user clicks or taps the Character, THE App SHALL trigger a Glow_Animation consisting of a radial light pulse expanding outward from the Character's center.
2. THE Glow_Animation SHALL use colors consistent with the blue/violet/white palette of the App, with a peak opacity between 0.1 and 0.6.
3. THE Glow_Animation SHALL expand to no more than 150% of the Character's bounding-box diameter and SHALL fade out completely within 600ms of being triggered.
4. THE Glow_Animation SHALL render behind the Character layer so it does not obscure the Character during its execution.
5. WHEN a new Interaction occurs while a Glow_Animation is already in progress, THE App SHALL cancel the current Glow_Animation, reset it to its initial state, and restart it from the beginning, resetting the 600ms fade-out timer.

---

### Requirement 12: Particle Effect on Interaction

**User Story:** As a user, I want small floating particles to appear when I interact with the character, so that each click creates a delightful, magical visual moment.

#### Acceptance Criteria

1. WHEN an Interaction occurs, THE App SHALL spawn between 6 and 12 Particle_Effect elements positioned within 20px of the Character's bounding box center.
2. WHEN Particle_Effect elements are spawned, THE App SHALL animate each element upward and outward from its spawn point with a randomized direction between 0° and 360°, a randomized speed between 40px/s and 120px/s, and a randomized initial opacity between 0.6 and 1.0.
3. THE Particle_Effect elements SHALL fade from their initial opacity to 0 and be removed from the DOM within 1200ms of being spawned.
4. THE Particle_Effect elements SHALL render as one of the following shapes: a filled circle with diameter between 4px and 10px, or a star glyph with size between 8px and 14px, consistent with the night/moonlight theme.
5. THE Particle_Effect elements SHALL use colors drawn exclusively from the App's blue, violet, and white palette with no other colors applied.
6. WHEN multiple rapid Interactions occur within the 1200ms lifetime of existing Particle_Effect elements, THE App SHALL spawn new Particle_Effect elements without removing or interrupting the animation of existing in-flight particles, up to a maximum of 60 simultaneous Particle_Effect elements.

---

### Requirement 13: Character Expression Variation

**User Story:** As a user, I want the character's appearance to shift subtly on each interaction, so that the character feels alive rather than static.

#### Acceptance Criteria

1. THE App SHALL define at least two distinct SVG visual states for the Character: a neutral state (default, no interaction) and an active state (during interaction), where each state is distinguishable by at least one of the following: opacity value, element position offset, or glow intensity value.
2. WHEN an Interaction occurs, THE App SHALL apply a CSS class change or SVG attribute change to the Character within 16ms (one animation frame at 60fps) to reflect the active state.
3. WHEN the Interaction Scale_Animation completes, THE App SHALL begin transitioning the Character back to its neutral state, and the transition SHALL complete within 500ms.
4. IF the Character is already in the active state and a new Interaction occurs, THEN THE App SHALL reset the 500ms neutral-state transition timer without applying a duplicate state change.
5. THE Character expression or pose change SHALL be limited to: opacity shifts within a ±0.4 range of the neutral value, element position offsets of no more than 4px in any direction, or glow intensity changes of no more than 8px blur radius — and SHALL NOT alter the overall silhouette or recognizability of the Character.

---

### Requirement 14: Responsive Layout

**User Story:** As a user on any device, I want the app to display correctly and be fully usable, so that I can enjoy the experience on my phone, tablet, or desktop.

#### Acceptance Criteria

1. THE App SHALL render all UI elements without clipping, overlapping, or invisible text at viewport widths from 320px to 2560px, ensuring all visible content is legible without user-initiated zoom.
2. THE Character SHALL remain horizontally and vertically centered at all supported viewport sizes, with its bounding box midpoint no more than 2px from the viewport center.
3. THE App SHALL use Tailwind responsive utility classes to apply font sizes of at least 12px for body text and at least 28px for the main heading, padding of at least 8px on all sides for containers, and touch targets of at least 44×44px for interactive elements across all breakpoints.
4. THE App SHALL not display horizontal scrollbars at any supported viewport width.
5. WHEN the App is viewed on a touch-capable device, THE App SHALL respond to tap events on the Character and CTA_Button and initiate the Interaction sequence within 300ms of the tap end event.
6. THE Character and CTA_Button elements SHALL have `-webkit-tap-highlight-color: transparent` or equivalent applied so that no tap highlight color is visible on mobile browsers.
7. IF the viewport width is less than 320px, THEN THE App SHALL display a non-overlapping layout where all interactive elements remain accessible, with no loss of core functionality.

---

### Requirement 15: Accessibility

**User Story:** As a user relying on keyboard or assistive technology, I want to interact with the app, so that the experience is not exclusively pointer-dependent.

#### Acceptance Criteria

1. THE Character SHALL be focusable via keyboard Tab navigation, SHALL display a visible focus indicator with a minimum outline width of 2px, and SHALL trigger the Interaction sequence when the Enter or Space key is pressed while focused.
2. THE CTA_Button SHALL be a native `<button>` element or have `role="button"` and `tabindex="0"` so it is reachable by keyboard navigation, and IF the CTA_Button is not a native `<button>` element, THEN THE CTA_Button SHALL also respond to Enter and Space key press events to trigger its action.
3. THE App SHALL provide an `aria-label` on the Character element describing its interactive purpose (e.g., "Interact with Light Night").
4. THE Reaction_Area SHALL have an `aria-live="polite"` attribute and `aria-atomic="false"` so screen readers announce only the newly added Reaction_Quote content after each Interaction.
5. THE Counter_Component SHALL expose the current Interaction_Counter value to assistive technologies via an `aria-live="polite"` region or `aria-valuenow` attribute, and WHEN the Interaction_Counter value changes, THE Counter_Component SHALL ensure the updated value is announced by screen readers within 1 second of the change.
6. THE App SHALL maintain a minimum color contrast ratio of 4.5:1 between all text elements and their immediate backgrounds, as defined by WCAG 2.1 AA, and IF any text element has a font size of at least 18pt (24px) or 14pt (18.67px) bold, THEN THE App SHALL maintain a minimum contrast ratio of 3:1 for that text element.

---

### Requirement 16: Performance

**User Story:** As a user, I want the app to feel instantaneous and smooth, so that animations and interactions never feel laggy or janky.

#### Acceptance Criteria

1. THE App SHALL trigger the Scale_Animation, Glow_Animation, and Particle_Effect within one animation frame (≤16ms) of receiving an Interaction event.
2. THE App SHALL use CSS `transform` and `opacity` properties exclusively for animations — rather than properties that trigger layout reflow — to maintain 60fps rendering.
3. THE App SHALL remove Particle_Effect DOM elements within 100ms after their animation completes to prevent unbounded DOM growth.
4. WHEN loaded over a standard broadband connection (≥25 Mbps download), THE App SHALL reach an interactive state within 3 seconds of the initial HTML request.
5. IF the device reports a preference for reduced motion via the `prefers-reduced-motion` media query, THEN THE App SHALL disable Scale_Animation, Glow_Animation, and Particle_Effect and present equivalent non-motion feedback instead.
6. IF frame rendering time exceeds 16ms for more than 3 consecutive frames during any animation, THEN THE App SHALL cancel remaining Particle_Effect instances for the current Interaction event and complete Scale_Animation and Glow_Animation without spawning additional Particle_Effect elements.
