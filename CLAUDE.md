# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EVKit is a marketing website for an iOS EV (Electric Vehicle) cost calculator app launching March 2026. The site is a static HTML/CSS/JavaScript website designed with premium aesthetics, accessibility, and mobile-first responsiveness.

**Key Details:**
- Pure HTML/CSS/JS (no build process, dependencies: GSAP/ScrollTrigger via CDN, html2canvas)
- Hosted as static site (GitHub Pages compatible)
- Domain: evkit.app
- Price: £2.99 one-time purchase (no subscriptions)
- Company: Kukkdi Ltd

## File Structure

```
evkit_html/
├── index.html              # Main landing page with all sections
├── privacy.html            # Privacy policy page
├── terms.html             # Terms of service page
├── style.css              # Single stylesheet for all pages (v=39+ cache busting)
├── screenshot-generator.html  # Tool to generate placeholder screenshots
├── CNAME                  # Domain configuration for GitHub Pages
└── assets/
    ├── README.md          # Screenshot specifications and guidelines
    ├── hero-vehicle.png   # Hero section screenshot (Tesla Model Y)
    ├── calculate-manual.png    # Calculate section - manual mode
    ├── calculate-vehicle.png   # Calculate section - selected vehicle mode
    ├── compare.png        # Compare section screenshot
    ├── ev-comparison-sample.pdf  # Sample PDF export for Compare section
    ├── evkit_1_dark.png   # Battery Intelligence screenshots (dark + light)
    ├── evkit_1_light.png
    └── ...                # Additional carousel screenshots
```

## Design System

### Color Palette (CSS Variables in :root)
- `--primary-dark: #0a0a0a` - Primary background (very dark gray)
- `--elevated-dark: #141414` - Elevated sections (subtle alternation)
- `--card-dark: #1a1a1a` - Cards and elevated elements
- `--primary-mint: #62D2A2` - Brand accent (systemMint)
- `--primary-mint-light: #76E5B4` - Lighter mint variant
- `--mint-glow: rgba(98, 210, 162, 0.2)` - Mint glow effect

### Easing Functions
- `--ease-bounce: cubic-bezier(0.34, 1.56, 0.64, 1)` - Bouncy interactions
- `--ease-smooth: cubic-bezier(0.4, 0, 0.2, 1)` - Smooth transitions

### Typography
- Font family: `-apple-system, BlinkMacSystemFont, 'Segoe UI', ...` (system fonts)
- Hero headline: Two-tier structure with `.headline-primary` (3.5rem) and `.headline-secondary` (2.45rem at 70%)
- Gradient effect on highlighted text via `background-clip: text`

### Premium Design Patterns
- **Multi-layered shadows**: 3+ shadow layers for depth (ambient, directional, glow)
- **Alternating backgrounds**: Sections alternate between `--primary-dark` and `--elevated-dark` for subtle visual rhythm
- **Staggered animations**: Trust badges use `nth-child` delays for sequential floating
- **Shimmer effects**: Feature cards have `::before` pseudo-element shimmer on hover
- **GPU-accelerated animations**: Uses `transform` and `opacity` (not `left`/`top`)
- **Proportional scaling**: Screenshot border-radius (29px) matches PNG's baked corners (42px) at 69.2% scale

## Architecture Patterns

### 1. Screenshot Carousel System

**Structure**: Hero section right column contains a theme-switchable, navigable carousel.

**HTML Pattern**:
```html
<div class="screenshot-carousel">
  <div class="screenshot-slide active" data-slide="1">
    <img class="screenshot-image"
         src="assets/evkit_1_dark.png"
         data-dark="assets/evkit_1_dark.png"
         data-light="assets/evkit_1_light.png"
         alt="...">
  </div>
  <!-- Repeat for slides 2-6 -->
</div>
```

**Key JavaScript Functions** (in index.html `<script>`):
- `showSlide(slideNumber)` - Shows specific slide, updates indicators
- `switchTheme(theme)` - Switches between 'dark' and 'light' screenshots
- `nextSlide()` / `prevSlide()` - Navigation with looping
- Auto-advance every 4 seconds (pauses on hover/touch)
- Keyboard navigation (ArrowLeft/ArrowRight)
- Touch swipe support with 50px threshold

**Important**: Screenshots must have both `data-dark` and `data-light` attributes for theme switching to work.

### 2. Registration Modal System

**Purpose**: "Get Early Access to EVKit" button opens modal that generates a pre-filled mailto: link.

**Flow**:
1. User clicks CTA button → `openModal()` called
2. Modal shows with focus trap (Tab cycles through form elements)
3. Form validates name (optional) + preference (required radio buttons)
4. On submit → generates `mailto:hello@evkit.app` with pre-filled subject/body
5. Opens email client → closes modal after 500ms

**Accessibility Features**:
- Focus trap (`trapFocus()` function prevents Tab escaping modal)
- Escape key closes modal
- Focus restoration (returns to previously focused element)
- Click outside modal closes it
- ARIA labels on all interactive elements

**Email Preferences**:
- `beta` → "Interested in beta testing"
- `notify` → "Just notify me when the app goes live"
- `both` → "Both - beta test AND notify at launch"

### 3. Calculate Section (Know Before You Go)

**Purpose**: Shows journey cost calculator with mode switching between manual input and selected vehicle.

**HTML Pattern**:
```html
<div class="calculate-tabs">
    <button class="calculate-tab active" data-mode="manual">Manual Calc</button>
    <button class="calculate-tab" data-mode="vehicle">Selected Vehicle</button>
</div>
<div class="device-iphone calculate-phone">
    <img src="assets/calculate-manual.png"
         data-manual="assets/calculate-manual.png"
         data-vehicle="assets/calculate-vehicle.png"
         class="screen-screenshot calculate-screenshot">
</div>
```

**JavaScript**: Click handler switches `img.src` based on `data-mode` attribute and updates active tab styling.

**CSS Classes**:
- `.calculate-tabs` - Flex container for tab buttons (centered)
- `.calculate-tab` - Tab button with border styling
- `.calculate-tab.active` - Mint border and subtle glow
- `.calculate-showcase` - Main container (flex, centers phone + features)
- `.calculate-features` - Feature list beside/below phone

### 4. Winter Mode Section (Real Costs)

**Purpose**: Interactive CSS-only recreation of the app's winter mode controls with "exploded UI" animation effect.

**HTML Pattern**:
```html
<div class="winter-controls">
    <div class="winter-control winter-toggle">
        <span class="control-label">Enable winter mode</span>
        <div class="toggle-switch active"><div class="toggle-knob"></div></div>
    </div>
    <div class="winter-control winter-heatpump">
        <span class="control-label">Heat pump</span>
        <div class="toggle-switch"><div class="toggle-knob"></div></div>
    </div>
    <div class="winter-control winter-severity">
        <span class="control-label">Severity</span>
        <div class="segmented-control">
            <button class="segment">Mild</button>
            <button class="segment active">Moderate</button>
            <button class="segment">Severe</button>
        </div>
    </div>
    <div class="winter-control winter-result">
        <span class="result-value">+22%</span>
        <span class="result-label">efficiency loss</span>
    </div>
</div>
```

**GSAP Animation**: Controls fly in from different directions with rotation (`back.out` easing) creating an "exploded UI assembly" effect on scroll.

**Interactive JavaScript**: Toggle switches and segmented controls are clickable, updating the efficiency percentage dynamically.

**Efficiency Table**: Shows real-world efficiency data with/without heat pump across severity levels.

### 5. Compare Section (Head to Head)

**Purpose**: Shows vehicle comparison feature with PDF export capability.

**HTML Pattern**:
```html
<div class="compare-visual">
    <a href="assets/ev-comparison-sample.pdf" target="_blank" class="pdf-badge">
        <svg>...</svg>
        <span>Export as PDF</span>
        <span class="pdf-preview-hint">Preview</span>
    </a>
    <div class="device-iphone compare-phone">
        <img src="assets/compare.png" class="screen-screenshot">
    </div>
</div>
```

**PDF Badge**: Positioned overlay that links to sample PDF export. Styled with mint border, hover glow effect.

### 6. CSS Section Organization

The `style.css` file is organized into clearly marked sections:
1. **Reset & Base Styles** - CSS variables, resets, body defaults
2. **Skip to Main Content** - Accessibility skip link
3. **Focus States** - Keyboard navigation outlines
4. **Motion Preferences** - `prefers-reduced-motion` media query
5. **Navigation Header** - Sticky nav with logo and links
6. **Trust Bar** - Floating trust badges with staggered animation
7. **Hero Section** - Two-column layout with iPhone mockup
8. **Calculate Section** - Mode tabs, phone mockup, feature list
9. **Winter Mode Section** - Interactive controls, efficiency table
10. **Compare Section** - PDF badge overlay, phone mockup
11. **Battery Intelligence Section** - Tab switching, degradation/health screenshots
12. **Watch Section** - Apple Watch mockup placeholder
13. **CTA Section** - Final call-to-action
14. **Footer** - Site footer
15. **Modal Styles** - Registration form modal
16. **Animations** - `@keyframes` definitions (float, shimmer)
17. **Responsive Design** - Media queries for 968px, 768px, 640px, 480px breakpoints

## Critical Implementation Details

### Screenshot Aspect Ratio & Scaling
- **Aspect Ratio**: 9:19.5 (iPhone)
- **Recommended Source**: 1170 x 2532px (iPhone 14 Pro)
- **Display Size**: 270px wide (scaled from 390px)
- **Border Radius**: 29px (calculated as 42px × 270/390 = 42px × 0.692)
- **PNG Corners**: Baked into PNG using CSS `clip-path: inset(0 round 42px)` in screenshot-generator.html

**Why this matters**: The border-radius must match the PNG's baked corners proportionally, otherwise hover shadows reveal transparent corners.

### CSS Device Mockups
The site uses CSS-only device mockups for iPhones and Apple Watch:

**iPhone Pattern**:
```html
<div class="device-iphone">
    <div class="iphone-notch"></div>
    <div class="iphone-screen">
        <img src="assets/screenshot.png" class="screen-screenshot">
    </div>
</div>
```

**Key CSS Classes**:
- `.device-iphone` - Main container with rounded corners, border, aspect ratio
- `.iphone-notch` - Dynamic Island notch at top
- `.iphone-screen` - Screen area containing screenshot
- `.screen-screenshot` - Full-width screenshot image

**Variants**: `.hero-phone`, `.calculate-phone`, `.compare-phone`, `.battery-phone` for section-specific sizing.

### Alternating Section Backgrounds
Pattern learned from flipunit.app analysis. Sections alternate between `--primary-dark` (#0a0a0a) and `--elevated-dark` (#141414):

1. **Hero** - `--primary-dark`
2. **Calculate** (Know Before You Go) - `--elevated-dark`
3. **Winter Mode** (Real Costs) - `--primary-dark`
4. **Compare** (Head to Head) - `--elevated-dark`
5. **Battery Intelligence** - `--primary-dark`
6. **Watch** - `--elevated-dark`
7. **CTA** - `--primary-dark`
8. **Footer** - `--elevated-dark`

This creates subtle visual separation between sections (only 10-unit hex difference).

### Mobile Carousel Arrows
Navigation arrows remain visible on mobile (initially were hidden). CSS previously had `.hero-nav-btn { display: none; }` in mobile media query - this was removed to maintain navigation on all screen sizes.

### Two-Tier Headline Pattern
```html
<h1 class="hero-headline">
  <span class="headline-primary">The <span class="highlight">EV</span> Cost Calculator</span>
  <span class="headline-secondary">That Gets It Right</span>
</h1>
```
- Primary line uses bold weight (700), larger size (3.5rem)
- Secondary line is 70% size (2.45rem), lighter weight (500), lighter color (#a1a1a6)
- Gradient applied to `<span class="highlight">` within primary line

### Cache Busting
All pages load `style.css?v=39` (or higher) - increment version parameter when updating styles to bust browser/CDN cache.

### GSAP ScrollTrigger Animations
The site uses GSAP with ScrollTrigger plugin for scroll-based animations:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js"></script>
```

**Animation Patterns**:
- **Fade up**: Elements animate from `y: 30` to final position
- **Slide in**: Elements slide from `x: -30` or `x: 30`
- **Exploded UI**: Winter Mode controls fly in from different directions with rotation
- **Stagger**: Multiple elements animate sequentially with `stagger: 0.1` or similar

**ScrollTrigger Config**:
```javascript
scrollTrigger: {
    trigger: '.section-class',
    start: "top 85%",
    toggleActions: "play none none reverse"
}
```

## Common Editing Scenarios

### Adding New Screenshots
1. Generate PNG at 1170 x 2532px (or use screenshot-generator.html)
2. Create both dark and light versions
3. Name as `evkit_N_dark.png` and `evkit_N_light.png` (where N = slide number)
4. Add to index.html carousel following existing pattern
5. Ensure both `data-dark` and `data-light` attributes are set
6. Update `totalSlides` count will auto-update via `screenshots.length`

### Modifying Hero Copy
**Headline**: Lines 54-57 in index.html (two-tier structure)
**Subheadline**: Lines 58-60 (emphasizes "real data — not guesses")
**CTA Button**: Line 62 ("Get Early Access to EVKit")
**Tagline**: Line 61 (pricing and launch date)

### Updating Colors
Edit CSS variables in `:root` (style.css lines 5-16). Changes cascade throughout site due to `var(...)` references.

### Adding Feature Cards
Features use a CSS Grid layout (`.features-grid`). Each card follows this structure:
```html
<div class="feature-card">
  <div class="feature-icon">[SVG icon]</div>
  <h3 class="feature-title">Title</h3>
  <p class="feature-description">Description</p>
</div>
```
Hover effects (scale, rotate, shimmer) apply automatically via CSS.

### Adjusting Responsive Breakpoints
Two main breakpoints:
- **768px** (tablets): Hero becomes single column, reduces font sizes
- **480px** (mobile): Further size reductions, increased spacing

## Accessibility Compliance

This site follows WCAG 2.1 AA standards:
- Skip to main content link (`.skip-link`)
- Keyboard navigation with visible focus states (3px outline)
- `aria-label` attributes on icon-only buttons
- Focus trap in modal with Escape key support
- `prefers-reduced-motion` respect (animations reduced to 0.01ms)
- Semantic HTML (`<header>`, `<nav>`, `<main>`, `<section>`, `<footer>`)
- Alt text on all images
- Color contrast ratios meet AA standards (mint on dark backgrounds)

## Email Contact Pattern

All email links use this format:
```html
mailto:hello@evkit.app?subject=EVKit:%20[Context]
```
- Subject prefix: "EVKit: " (with space after colon)
- URL-encoded subject and body parameters
- Modal form generates pre-filled emails (name + preference)

## Screenshot Generator Tool

`screenshot-generator.html` is a standalone utility for creating placeholder screenshots:
- Uses html2canvas to render mockup → PNG
- Generates 1170 x 2532px images (iPhone 14 Pro resolution)
- Applies `clip-path: inset(0 round 42px)` for baked rounded corners
- Outputs dark and light variants
- **Not linked from main site** - internal tool only

## Legal Pages

`privacy.html` and `terms.html` share:
- Same header/footer as index.html
- Special `.privacy-content` wrapper for legal text styling
- Last updated date: January 26, 2026
- Company: Kukkdi Ltd
- Contact: hello@evkit.app

Both pages emphasize:
- 100% on-device calculations (no server processing)
- Optional iCloud sync via Apple CloudKit
- Anonymized analytics via TelemetryDeck
- No personal data collection or storage

## Testing Checklist

When making changes, verify:
1. **Hero**: iPhone mockup displays correctly with real screenshot
2. **Calculate Section**: Mode tabs switch between Manual Calc / Selected Vehicle screenshots
3. **Winter Mode**: Toggle switches and segmented controls are interactive, efficiency % updates
4. **Compare Section**: PDF badge links to sample PDF, phone mockup displays correctly
5. **Battery Intelligence**: Tab switching works for degradation/health screenshots
6. **GSAP Animations**: Scroll-triggered animations fire correctly (exploded UI effect, slide-ins)
7. **Modal**: Opens, closes (X, Escape, outside click), focus trap works
8. **Form**: Validates preference selection, generates correct mailto link
9. **Responsive**: Test at 968px, 768px, 640px, and 480px breakpoints
10. **Accessibility**: Tab navigation works, focus states visible, skip link appears on focus
11. **Mobile Alignment**: All sections centered properly on mobile devices
