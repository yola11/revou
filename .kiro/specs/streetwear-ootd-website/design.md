# Design Document: Streetwear OOTD Website

## Overview

The streetwear OOTD website is a single-page, immersive fashion editorial experience delivered as one `index.html` file. The site targets a luxury streetwear audience and is architecturally inspired by editorial campaign sites from Nike, Ader Error, and Awwwards-winning designs. All styling is handled by Tailwind CSS (CDN) with supplemental custom CSS properties, and all interactivity is implemented in embedded Vanilla JavaScript.

The page is structured as a sequence of full-width sections scrolled vertically, each with its own visual identity yet unified by the Dark_Palette (`#0a0a0a`, `#1a1a1a`, `#f5f5f0`, accent electric blue `#3b82f6` / soft purple `#8b5cf6`). Three global JavaScript systems — the Magnetic Button engine, the Mouse-Follow Glow, and the Scroll Animation Observer — are initialised once and applied declaratively via HTML data attributes, keeping each section's markup clean.

**Tech stack:**
- HTML5 (semantic elements)
- Tailwind CSS v3 via Play CDN (`<script src="https://cdn.tailwindcss.com">`)
- Tailwind config block (inline `<script>` with `tailwind.config`) for custom colors, fonts, and keyframes
- Vanilla JavaScript (ES2020, no external libraries)
- CSS custom properties (`:root`) for palette + animation tokens

---

## Architecture

### Single-File Structure

```
index.html
├── <head>
│   ├── <meta charset="UTF-8">
│   ├── <meta name="viewport" content="width=device-width, initial-scale=1.0">
│   ├── <title>
│   ├── Tailwind CDN <script src="https://cdn.tailwindcss.com">
│   ├── Tailwind config <script> (tailwind.config = { ... })
│   └── <style> block — CSS custom properties, keyframes, base resets, utility classes
│
├── <body>
│   ├── #page-transition  (overlay, fades out on load)
│   ├── #glow-cursor      (mouse-follow radial gradient layer, desktop only)
│   ├── <header> — Navbar
│   ├── <main>
│   │   ├── <section id="hero">         Hero Banner
│   │   ├── <section id="featured">     Featured Collection
│   │   ├── <section id="trending">     Trending OOTD
│   │   ├── <section id="new-arrivals"> New Arrivals
│   │   ├── <section id="gallery">      Style Inspiration Gallery
│   │   ├── <section id="story">        Brand Story
│   │   ├── <section id="reviews">      Customer Reviews
│   │   └── <section id="newsletter">   Newsletter
│   └── <footer id="footer">            Premium Footer
│
└── <script> — embedded JS modules (IIFE-wrapped, ES2020)
    ├── PageTransition
    ├── NavbarController
    ├── ParallaxEngine
    ├── MagneticButton
    ├── MouseGlow
    ├── ScrollAnimationObserver
    ├── CarouselController
    └── NewsletterForm
```

### Rendering Model

The page is entirely static HTML rendered by the browser with no server-side rendering or build step. Content (products, reviews, gallery images) is declared as JavaScript data arrays in the embedded `<script>` block and injected into the DOM at `DOMContentLoaded`. This pattern keeps HTML templates minimal and data easy to edit.

---

## Components and Interfaces

### 1. PageTransition

**Element:** `<div id="page-transition">`  
**Behaviour:** Covers the full viewport on load, then fades out via CSS animation over 1.2 s.

```
PageTransition
  └── init(): void
        → sets animation-duration: 1.2s
        → on animationend: element.style.display = 'none'
```

### 2. NavbarController

**Element:** `<header id="navbar">`

```
NavbarController
  ├── init(): void
  │     → attaches scroll listener
  │     → attaches click listeners to nav links
  │     → attaches click listener to hamburger button
  ├── handleScroll(scrollY: number): void
  │     → if scrollY > 50: classList.add('scrolled')   → applies glassmorphism
  │     → else: classList.remove('scrolled')
  ├── handleNavClick(anchor: string): void
  │     → document.querySelector(anchor).scrollIntoView({ behavior: 'smooth' })
  └── toggleMobileMenu(): void
        → classList.toggle('menu-open') on overlay
        → triggers staggered animation on mobile nav links
```

**CSS classes:**
- `.scrolled` → `backdrop-filter: blur(12px)`, `background: rgba(10,10,10,0.75)`, `border-bottom: 1px solid rgba(255,255,255,0.08)`

### 3. ParallaxEngine

**Target:** Hero background layer `#hero-bg`

```
ParallaxEngine
  ├── init(): void
  │     → attaches scroll listener (passive)
  └── handleScroll(scrollY: number): void
        → bgOffset = scrollY * 0.5   (50% of foreground scroll)
        → heroBackground.style.transform = `translateY(${bgOffset}px)`
```

The hero foreground (text, CTA) stays at natural scroll speed. Background translates at 0.4–0.6× the scroll offset, implemented as `scrollY * 0.5` (midpoint of 40–60% range).

### 4. MagneticButton

**Targets:** All elements with `[data-magnetic]` attribute

```
MagneticButton
  ├── init(): void
  │     → if window.innerWidth < 768: return (disabled on mobile)
  │     → querySelectorAll('[data-magnetic]') → attach listeners to each
  ├── handleMouseMove(button: Element, e: MouseEvent): void
  │     → rect = button.getBoundingClientRect()
  │     → cx = rect.left + rect.width / 2
  │     → cy = rect.top + rect.height / 2
  │     → dx = e.clientX - cx
  │     → dy = e.clientY - cy
  │     → dist = Math.hypot(dx, dy)
  │     → PROXIMITY_RADIUS = 50  (midpoint of 40–60px range)
  │     → if dist < PROXIMITY_RADIUS:
  │           SHIFT = 0.275  (midpoint of 20–35% range)
  │           tx = dx * SHIFT
  │           ty = dy * SHIFT
  │           button.style.transform = `translate(${tx}px, ${ty}px)`
  └── handleMouseLeave(button: Element): void
        → button.style.transform = 'translate(0, 0)'
        → button.style.transition = 'transform 350ms ease-out'
```

**Performance:** Mousemove handler uses `requestAnimationFrame` wrapper — the rAF flag is set on each call and cleared in the rAF callback, preventing queuing of more than one pending frame per element.

### 5. MouseGlow

**Element:** `<div id="glow-cursor">`

```
MouseGlow
  ├── init(): void
  │     → if window.innerWidth < 768: return (disabled on mobile)
  │     → attaches mousemove listener on document
  └── handleMouseMove(e: MouseEvent): void
        → glowEl.style.left = `${e.clientX}px`
        → glowEl.style.top  = `${e.clientY}px`
```

**CSS for `#glow-cursor`:**
```css
#glow-cursor {
  position: fixed;
  width: 600px; height: 600px;
  border-radius: 50%;
  pointer-events: none;
  transform: translate(-50%, -50%);
  background: radial-gradient(circle, rgba(59,130,246,0.10) 0%, transparent 70%);
  transition: left 100ms linear, top 100ms linear;
  z-index: 0;
}
```

### 6. ScrollAnimationObserver

**Targets:** All elements with `[data-animate]` attribute  
**Staggered children:** Elements with `[data-stagger-parent]` — direct children receive incremental `animation-delay`.

```
ScrollAnimationObserver
  ├── init(): void
  │     → observer = new IntersectionObserver(callback, { threshold: 0.15 })
  │     → querySelectorAll('[data-animate]') → observe each
  └── callback(entries): void
        → for each entry where isIntersecting:
              entry.target.classList.add('is-visible')
              if entry.target has [data-stagger-parent]:
                children.forEach((el, i) → el.style.animationDelay = `${i * 120}ms`)
              observer.unobserve(entry.target)
```

**CSS (initial hidden state — always in CSS, never JS):**
```css
[data-animate] {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 700ms ease-out, transform 700ms ease-out;
}
[data-animate].is-visible {
  opacity: 1;
  transform: translateY(0);
}
```

### 7. CarouselController

**Targets:** `#reviews-carousel`  
Used only by the Customer Reviews section to advance/retreat slides.

```
CarouselController
  ├── init(carouselEl: Element, reviewData: Review[]): void
  │     → renders review cards into carouselEl
  │     → attaches prev/next arrow click listeners
  │     → if reviewData.length > 3: renders navigation dots
  ├── goTo(index: number): void
  │     → updates activeIndex
  │     → applies translateX to slide track
  └── renderDots(): void
        → creates dot elements equal to reviewData.length
        → marks active dot
```

### 8. NewsletterForm

```
NewsletterForm
  ├── init(): void
  │     → attaches submit listener on #newsletter-form
  ├── validate(email: string): boolean
  │     → regex: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  │     → returns true if match, false otherwise
  ├── handleSubmit(e: Event): void
  │     → e.preventDefault()
  │     → if validate(input.value):
  │           showSuccess()
  │     → else:
  │           showError()
  ├── showSuccess(): void
  │     → hide form, fade-in success message (300ms)
  └── showError(): void
        → display inline error adjacent to input (no page reload)
```

---

## CSS Architecture

### Custom Properties (`:root`)

```css
:root {
  /* Palette */
  --color-bg:        #0a0a0a;
  --color-surface:   #1a1a1a;
  --color-text:      #f5f5f0;
  --color-text-muted:#9ca3af;
  --color-accent-blue:  #3b82f6;
  --color-accent-purple:#8b5cf6;

  /* Spacing tokens */
  --section-padding: clamp(4rem, 8vw, 8rem);

  /* Typography tokens */
  --font-display: 'Inter', 'Helvetica Neue', sans-serif;
  --font-size-hero: clamp(4rem, 10vw, 9rem);
  --font-size-section: clamp(2.5rem, 6vw, 5rem);

  /* Animation tokens */
  --anim-duration-reveal: 700ms;
  --anim-ease-out: cubic-bezier(0.16, 1, 0.3, 1);
  --anim-stagger-base: 120ms;
  --transition-glow: 100ms linear;
  --transition-magnetic-leave: 350ms ease-out;
}
```

### Tailwind Config Block

```html
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          'bg-base':     '#0a0a0a',
          'bg-surface':  '#1a1a1a',
          'off-white':   '#f5f5f0',
          'accent-blue': '#3b82f6',
          'accent-purple':'#8b5cf6',
        },
        fontFamily: {
          display: ['Inter', 'Helvetica Neue', 'sans-serif'],
        },
        animation: {
          'gradient-shift': 'gradientShift 8s ease infinite',
          'fade-in-overlay': 'fadeInOverlay 1.2s ease forwards',
          'pulse-bounce':    'pulseBounce 2s ease-in-out infinite',
          'badge-new-drop':  'badgePulse 1.5s ease-in-out infinite',
        },
        keyframes: {
          gradientShift: {
            '0%, 100%': { backgroundPosition: '0% 50%' },
            '50%':       { backgroundPosition: '100% 50%' },
          },
          fadeInOverlay: {
            '0%':   { opacity: '1' },
            '100%': { opacity: '0', pointerEvents: 'none' },
          },
          pulseBounce: {
            '0%, 100%': { transform: 'translateY(0)' },
            '50%':      { transform: 'translateY(8px)' },
          },
          badgePulse: {
            '0%, 100%': { opacity: '1',   transform: 'scale(1)' },
            '50%':      { opacity: '0.7', transform: 'scale(1.05)' },
          },
        },
      },
    },
  }
</script>
```

### Glassmorphism Utility Class

```css
.glass {
  background: rgba(10, 10, 10, 0.75);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.08);
}
```

Applied to: Navbar (when `.scrolled`), Customer Review cards.

### Layering Strategy

| Layer | z-index | Element |
|-------|---------|---------|
| Glow cursor | 0 | `#glow-cursor` |
| Page content | 1–10 | All sections |
| Navbar | 50 | `<header>` |
| Mobile overlay menu | 60 | `.mobile-menu` |
| Page transition | 100 | `#page-transition` |

---

## Data Models

All content is defined as typed JavaScript data arrays in the embedded `<script>` block. The render functions consume these arrays and generate DOM nodes.

### Product (Featured Collection & New Arrivals)

```js
/**
 * @typedef {Object} Product
 * @property {string} id          - Unique slug, e.g. "bomber-01"
 * @property {string} name        - Display name in all-caps, e.g. "ARCHIVE BOMBER"
 * @property {string} category    - Category label, e.g. "SS25 / OUTERWEAR"
 * @property {string} price       - Display price string, e.g. "$320"
 * @property {string} imageSrc    - URL or path to product image
 * @property {string} imageAlt    - Descriptive alt text
 * @property {boolean} isNew      - Shows "NEW DROP" badge if true
 * @property {'large'|'medium'|'small'} size - Controls asymmetric grid slot
 */
const FEATURED_PRODUCTS = [
  { id: 'bomber-01', name: 'ARCHIVE BOMBER', category: 'SS25 / OUTERWEAR',
    price: '$320', imageSrc: 'https://images.unsplash.com/...', imageAlt: 'Archive bomber jacket in black',
    isNew: false, size: 'large' },
  // ...min 4 items
];

const NEW_ARRIVALS = [
  { id: 'cargo-01', name: 'UTILITY CARGO', category: 'SS25 / BOTTOMS',
    price: '$180', imageSrc: 'https://images.unsplash.com/...', imageAlt: 'Utility cargo pants in charcoal',
    isNew: true, size: 'large' },
  // ...min 4 items
];
```

### OOTDCard (Trending OOTD)

```js
/**
 * @typedef {Object} OOTDCard
 * @property {string} id
 * @property {string} title       - e.g. "MONOCHROME VOID"
 * @property {string} descriptor  - Short line, e.g. "All-black utility layering"
 * @property {string} imageSrc
 * @property {string} imageAlt
 * @property {string} shopLabel   - Overlay CTA text, e.g. "Shop the Look"
 */
const OOTD_CARDS = [
  { id: 'ootd-01', title: 'MONOCHROME VOID', descriptor: 'All-black utility layering',
    imageSrc: '...', imageAlt: 'Model in all-black layered streetwear look',
    shopLabel: 'Shop the Look' },
  // ...min 3 items
];
```

### GalleryImage (Style Inspiration Gallery)

```js
/**
 * @typedef {Object} GalleryImage
 * @property {string} id
 * @property {string} imageSrc
 * @property {string} imageAlt
 * @property {string} caption     - Short overlay caption, e.g. "Graphic tee + wide leg"
 * @property {'tall'|'wide'|'square'} span - Controls mosaic grid area
 */
const GALLERY_IMAGES = [
  { id: 'gal-01', imageSrc: '...', imageAlt: 'Editorial shot of layered streetwear look',
    caption: 'Graphic tee + wide leg', span: 'tall' },
  // ...min 6 items
];
```

### Review (Customer Reviews)

```js
/**
 * @typedef {Object} Review
 * @property {string} id
 * @property {string} name        - Reviewer display name
 * @property {number} rating      - Integer 1–5
 * @property {string} text        - Review body text
 * @property {string} [location]  - Optional city/country
 */
const REVIEWS = [
  { id: 'rev-01', name: 'Jordan M.', rating: 5,
    text: 'Quality is insane. The bomber fits exactly like the editorial.', location: 'NYC' },
  // ...min 3 items
];

/** Aggregate computed from REVIEWS array */
const AGGREGATE_RATING = {
  score: 4.9,    // average of all ratings
  count: 128,    // total review count (can include off-page reviews)
};
```

---

## Interaction Design Specifications

### Section Layout Details

#### Hero Banner (`#hero`)
- Full viewport: `min-height: 100vh`, `position: relative`, `overflow: hidden`
- Two-layer stacking: `#hero-bg` (absolutely positioned, covers full section, `will-change: transform`) and `#hero-fg` (relative, `z-index: 1`, flex column center)
- Background: dark gradient + subtle `AnimatedGradient` cycling between `#3b82f6` at 4% opacity and `#8b5cf6` at 4% opacity
- Headline: `font-size: clamp(4rem, 10vw, 9rem)`, `font-weight: 900`, `letter-spacing: -0.03em`, `line-height: 0.9`
- Sub-copy: `font-size: clamp(0.875rem, 2vw, 1.125rem)`, `color: var(--color-text-muted)`
- Scroll indicator: `<div class="scroll-indicator">` — arrow-down SVG with `animate-pulse-bounce`

#### Featured Collection (`#featured`)
- Desktop grid: CSS Grid `grid-template-columns: 2fr 1fr 1fr` with one item spanning two rows for the feature item
- Mobile (`< 768px`): Two-column grid `grid-template-columns: 1fr 1fr`
- Eyebrow text: `position: absolute` or `margin-bottom: -1rem` relative to grid, `letter-spacing: 0.2em`, `font-size: 0.7rem`
- Item card: `overflow: hidden` on wrapper, `<img>` with `transition: transform 400ms ease` — hover applies `scale(1.08)`

#### Trending OOTD (`#trending`)
- Desktop: CSS Grid with explicit template areas creating overlapping/masonry feel — rows of differing heights via `grid-row: span 2` on one card
- Mobile: `display: flex; overflow-x: auto; scroll-snap-type: x mandatory` — each card `scroll-snap-align: start; min-width: 80vw`
- Heading: `font-size: clamp(3rem, 8vw, 7rem)`, `position: relative; z-index: 2` — negative margin-bottom pulls it over the first card
- Overlay: absolutely positioned, `opacity: 0`, `transition: opacity 300ms ease` — hover on card sets `opacity: 1`

#### New Arrivals (`#new-arrivals`)
- Grid: `grid-template-columns: 2fr 1fr 1fr` with first item `grid-row: span 2`
- "NEW DROP" badge: `position: absolute; top: 1rem; left: 1rem` — pulsing outline badge with `animate-badge-new-drop`
- CTA: `[data-magnetic]` button, same style as Hero CTA

#### Style Inspiration Gallery (`#gallery`)
- Desktop mosaic: CSS Grid with named areas — example 3×3 grid where some cells span 2 columns or 2 rows
- Mobile (`< 640px`): `grid-template-columns: 1fr 1fr; grid-auto-rows: 240px` — uniform two-column
- Hover overlay: `position: absolute; inset: 0; background: rgba(0,0,0,0.5); opacity: 0; transition: opacity 350ms ease` + caption text

#### Brand Story (`#story`)
- Full-width: `width: 100%; min-height: 80vh`
- Background: Either `<img>` with `object-fit: cover` + dark overlay `rgba(10,10,10,0.65)`, or `AnimatedGradient` background
- Text contrast: off-white `#f5f5f0` on dark overlay ensures ≥ 4.5:1 ratio (verified against WCAG AA)
- Accent element: a `<blockquote>` styled with oversized `"` glyph and `font-size: clamp(1.5rem, 4vw, 2.5rem)`

#### Customer Reviews (`#reviews`)
- Desktop: CSS Grid `grid-template-columns: repeat(3, 1fr)`
- Mobile: Carousel — single card visible, prev/next arrows, dot indicators
- Card: `.glass` class + `border-radius: 1rem; padding: 1.5rem`
- Aggregate: flex row `★ 4.9 / 5 · 128 reviews` above the card grid/carousel

#### Newsletter (`#newsletter`)
- Background: `background-size: 200% 200%; animation: gradientShift 8s ease infinite` cycling between accent blue and accent purple at ~15% opacity on dark base
- Form: `display: flex; gap: 0.5rem` on desktop, stacked on mobile
- Success/error state: transition handled by `opacity` and `max-height` to avoid layout shift

#### Premium Footer (`#footer`)
- Desktop: `display: grid; grid-template-columns: 2fr 1fr 1fr 1fr` (brand column wider)
- Mobile (`< 640px`): `grid-template-columns: 1fr`; link columns use `<details>` + `<summary>` accordion for compactness
- Back-to-top: `<button>` fixed bottom-right OR inline at footer top — `window.scrollTo({ top: 0, behavior: 'smooth' })`
- Top border: `border-top: 1px solid rgba(255,255,255,0.08)`

---

## Performance Considerations

### GPU Compositing
- `will-change: transform` is added to `#hero-bg` (parallax), all `[data-animate]` elements while animating (removed via JS on `transitionend`), and `#glow-cursor`.
- Applied only to elements actively animating — not as a blanket style.

### Lazy Loading
- All `<img>` elements outside the Hero Banner carry `loading="lazy"`.
- Hero banner image (if used) carries `fetchpriority="high"` to hint the browser to pre-load it.

### Event Listener Optimisation
- Scroll listeners use `{ passive: true }` to avoid blocking the main thread.
- The MagneticButton mousemove handler uses a rAF guard — at most one pending frame per button at any time.
- The MouseGlow mousemove uses CSS `transition` (100ms) on the element itself rather than JS interpolation, keeping position updates cheap.

### Animation Budget
- All scroll-triggered animations use `opacity` and `transform` only (compositor-safe properties).
- No `width`, `height`, `top`, `left`, or `box-shadow` animations that would trigger layout/paint.

### Image Strategy
- All product and gallery images sourced from Unsplash CDN (or similar) with width query params (`?w=800&q=75`) to avoid serving full-resolution files.
- `<img>` elements use `width` and `height` attributes to reserve layout space and avoid CLS (Cumulative Layout Shift).

### Scroll Performance
- IntersectionObserver callbacks unobserve elements after the first trigger — preventing repeated callbacks on re-entry.
- The parallax effect is limited to the Hero Banner only to avoid multiple simultaneous scroll calculations.

---

## Accessibility Considerations

### Contrast
- All body text uses `#f5f5f0` on `#0a0a0a` background → contrast ratio ≈ 19:1 (far exceeds WCAG AA 4.5:1).
- Muted text `#9ca3af` on `#0a0a0a` → ratio ≈ 5.9:1 (passes AA for normal text).
- Brand Story overlay: `#f5f5f0` on `rgba(10,10,10,0.65)` over dark image → verified ≥ 4.5:1 by maintaining a minimum overlay opacity of 0.65.
- Glassmorphism card text: off-white on `rgba(10,10,10,0.75)` frosted background → ratio ≥ 4.5:1.

### Keyboard Navigation
- Navbar links and hamburger button are `<button>` or `<a>` elements — natively focusable.
- Carousel prev/next arrows are `<button>` elements with `aria-label="Previous review"` / `"Next review"`.
- All interactive elements have visible `:focus-visible` outlines (2px solid `var(--color-accent-blue)`, 2px offset).
- Mobile menu overlay traps focus within itself while open (using a focus trap loop in JS).

### Semantic HTML
- Page structure: `<header>`, `<main>`, `<footer>` as top-level landmarks.
- Each page section uses `<section>` with an `aria-labelledby` pointing to the section heading.
- Product cards use `<article>` elements.
- Review cards use `<article>` elements.
- Star ratings use `aria-label="5 out of 5 stars"` on the visual star element.
- Decorative images (gradient overlays, background images) use `role="presentation"` or `aria-hidden="true"`.

### Motion Preferences
- All scroll animations respect `prefers-reduced-motion`:
```css
@media (prefers-reduced-motion: reduce) {
  [data-animate] {
    transition: none;
    opacity: 1;
    transform: none;
  }
  .scroll-indicator { animation: none; }
  #page-transition  { animation: none; display: none; }
}
```
- JS checks `window.matchMedia('(prefers-reduced-motion: reduce)').matches` before attaching the ParallaxEngine and MagneticButton listeners.

### Touch Device Considerations
- MouseGlow and MagneticButton are disabled on `window.innerWidth < 768` (typically touch devices).
- The Trending OOTD horizontal scroll uses native `overflow-x: auto` with `scroll-snap-type` — works natively with touch swipe.
- Tap targets are minimum 44×44px (WCAG 2.5.5 AAA / 2.5.8 AA) for all buttons and nav links.

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After prework analysis, the following properties were identified as testable. Redundant checks were consolidated:

- Properties 4.4, 5.4, 6.4, 7.4, 8.3, 9.3 all test the same scroll animation pattern ("when element enters viewport, is-visible class is added with staggered delays") and are unified into **Property 1** (universal scroll animation).
- Properties 13.1 and 13.2 together cover the full magnetic button shift/return cycle — kept as **Property 2** (shift) and **Property 3** (return).
- Properties 12.1 and 12.4 cover glow positioning and mobile disabling — kept as separate properties since they test different logic.
- Properties 15.2 and 15.5 (lazy loading and alt attributes) are both per-element image properties — kept separate.
- Properties 2.2 and 2.4 (navbar scroll state and nav link click behaviour) are distinct logic paths — kept separate.
- Properties 10.3 / 10.4 are two sides of the same email validation logic — unified into **Property 8**.

### Property 1: Scroll-triggered class application with staggered delay

*For any* element marked `[data-animate]` that crosses the IntersectionObserver threshold (0.15), the `is-visible` CSS class shall be added to that element. For any parent element marked `[data-stagger-parent]`, each direct animated child shall receive an `animation-delay` that is 80–150 ms greater than the previous child's delay.

**Validates: Requirements 4.4, 5.4, 6.4, 7.4, 8.3, 9.3, 14.1, 14.3, 14.4**

### Property 2: Magnetic button shift within proximity

*For any* Magnetic_Button and any cursor position whose distance from the button center is less than the proximity radius (40–60px), the button's CSS transform translate values shall be between 20% and 35% of the cursor-to-center offset vector — and the shifted position shall always be closer to the cursor than the original position.

**Validates: Requirements 13.1**

### Property 3: Magnetic button return on departure

*For any* Magnetic_Button that has been shifted from its original position, when the cursor moves outside the proximity radius, the button's transform shall reset to `translate(0, 0)` and the transition shall complete within 400ms using ease-out timing.

**Validates: Requirements 13.2**

### Property 4: Mouse glow position tracks cursor

*For any* mouse coordinates `(x, y)` dispatched via a `mousemove` event on a desktop viewport (width ≥ 768px), the `#glow-cursor` element's `left` and `top` CSS values shall equal `x` and `y` respectively (within the CSS transition lag of ≤ 120ms).

**Validates: Requirements 12.1, 12.3**

### Property 5: Mouse glow and magnetic button disabled on mobile

*For any* viewport width less than 768px, the Mouse_Follow_Glow event listener shall not be attached (or the element shall have `display: none`) and no Magnetic_Button `mousemove` listener shall be active, such that cursor events produce no transform changes on magnetic elements.

**Validates: Requirements 12.4, 13.4**

### Property 6: Navbar scroll state responds to scroll position

*For any* scroll position greater than 50px from the page top, the navbar element shall carry the `scrolled` CSS class (applying glassmorphism styles). For any scroll position of 50px or less, the `scrolled` class shall be absent.

**Validates: Requirements 2.2**

### Property 7: Navigation links scroll to their target section

*For any* navigation link with an `href` value of `#<section-id>`, clicking that link shall cause the browser to scroll such that the element matching `document.querySelector('#<section-id>')` is brought into view via smooth scrolling behavior.

**Validates: Requirements 2.4**

### Property 8: Newsletter email validation state

*For any* string that matches the email pattern `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`, submitting the newsletter form shall replace the form with a success message (fade-in over 300ms) and no error message shall be displayed. For any string that does not match this pattern (including the empty string), the form shall display an inline error message and the form shall remain visible.

**Validates: Requirements 10.3, 10.4**

### Property 9: All non-hero images carry lazy loading attribute

*For any* `<img>` element in the document that is not a direct child of the `#hero` section, that element shall have the HTML attribute `loading="lazy"`.

**Validates: Requirements 15.2**

### Property 10: All images carry non-empty alt attributes

*For any* `<img>` element in the document, the `alt` attribute shall be present and shall contain a non-empty descriptive string.

**Validates: Requirements 15.5**

### Property 11: Stagger delay increments are within range

*For any* set of sibling elements with staggered animation delays generated by the ScrollAnimationObserver, the delay difference between each consecutive pair of elements shall be between 80ms and 150ms inclusive.

**Validates: Requirements 14.3**

### Property 12: Parallax background moves at 40–60% of scroll speed

*For any* scroll offset `Y` (in pixels), the `translateY` value applied to the hero background layer shall be between `0.40 × Y` and `0.60 × Y`.

**Validates: Requirements 3.3**

### Property 13: Content data completeness

*For any* rendered product card (Featured Collection or New Arrivals), the card shall contain an `<img>` element, a name text element, and a label element (category or price). *For any* rendered OOTD card, the card shall contain an `<img>` element, a title, and a descriptor. *For any* rendered review card, the card shall contain a name, a star rating element with an integer value between 1 and 5, and a review text element.

**Validates: Requirements 4.2, 5.1, 6.2, 9.1**

---

## Error Handling

### Newsletter Form Errors
- Empty input → inline message: *"Please enter your email address."*
- Invalid format → inline message: *"Please enter a valid email address."*
- Success → form replaced by: *"You're on the list."* (fade-in 300ms)
- All error/success messages are `role="alert"` with `aria-live="polite"` for screen reader announcement.

### Image Load Failures
- All `<img>` elements carry `onerror="this.src='data:image/svg+xml,...'"` pointing to a dark placeholder SVG matching the Dark_Palette, preventing broken image icons.

### IntersectionObserver Unsupported
- A feature check `if ('IntersectionObserver' in window)` gates the observer setup.
- Fallback: all `[data-animate]` elements immediately receive `is-visible` class via a synchronous loop, ensuring content is visible even without animation.

### MagneticButton Edge Cases
- If `getBoundingClientRect()` returns a zero-dimension rect (element not rendered), the handler returns early without applying transforms.
- Rapid mouse movement beyond proximity is handled gracefully — the `handleMouseLeave` always resets transform regardless of intermediate states.

### Carousel Navigation Bounds
- The carousel `goTo(index)` method clamps `index` to `[0, reviewData.length - 1]`, preventing out-of-bounds access.
- Prev/next buttons are disabled (`aria-disabled="true"`, `pointer-events: none`) when at the first/last slide respectively.

---

## Testing Strategy

### Overview

This feature is a static HTML page with embedded JavaScript logic. PBT is applicable to the pure logic layers (magnetic button math, scroll animation class application, email validation, parallax calculation, stagger delay generation) but not to the visual rendering, layout, or CSS-only behaviors.

**Property-based testing library:** [fast-check](https://github.com/dubzzz/fast-check) (JavaScript/TypeScript, runs in Node.js)

**Test runner:** [Vitest](https://vitest.dev/) (single execution: `vitest --run`)

### Property-Based Tests

Each of the 13 correctness properties maps to one property test. Tests are configured with a minimum of 100 iterations (`numRuns: 100`). Each test is tagged with a comment referencing the property number.

```
// Feature: streetwear-ootd-website, Property 2: Magnetic button shift within proximity
fc.assert(fc.property(
  fc.record({ cx: fc.float(), cy: fc.float(), ex: fc.float(), ey: fc.float() })
    .filter(({ cx, cy, ex, ey }) => Math.hypot(ex - cx, ey - cy) < 50),
  ({ cx, cy, ex, ey }) => {
    const { tx, ty } = computeMagneticShift(cx, cy, ex, ey, 50, 0.275);
    const ratio = Math.hypot(tx, ty) / Math.hypot(ex - cx, ey - cy);
    return ratio >= 0.20 && ratio <= 0.35;
  }
), { numRuns: 100 });
```

The logic under test (e.g., `computeMagneticShift`, `computeParallaxOffset`, `validateEmail`, `computeStaggerDelay`) is extracted as pure functions into a `utils.js` module, testable in isolation from the DOM.

### Unit Tests

Unit tests cover:
- Specific examples for `validateEmail` (valid addresses, edge cases like `a@b.c`, invalid patterns)
- Carousel `goTo()` bounds clamping — concrete examples with out-of-range indices
- `CarouselController` dot rendering — exactly N dots for N reviews
- `NavbarController.handleScroll(50)` vs `handleScroll(51)` boundary

### Integration / Structural Tests

Using [Playwright](https://playwright.dev/) or static HTML parsing (JSDOM):
- Verify single HTML file output with Tailwind CDN script tag
- Verify `<meta name="viewport">` presence
- Verify semantic elements (`<header>`, `<main>`, `<section>`, `<footer>`, `<nav>`)
- Verify minimum item counts (≥ 4 products, ≥ 3 OOTD cards, ≥ 6 gallery images, ≥ 3 reviews)
- Verify all `<img>` elements have non-empty `alt` and `loading="lazy"` (except hero)
- Verify CSS custom properties in `:root`
- Verify `backdrop-filter` on `.glass` class

### Responsive / Visual Tests

- Playwright viewport snapshots at 320px, 768px, 1024px, 1440px
- Verify no horizontal overflow at each breakpoint (using `document.body.scrollWidth <= viewportWidth`)

### Accessibility Audit

- [axe-core](https://github.com/dequelabs/axe-core) automated scan via Playwright to catch WCAG AA violations
- Manual check: keyboard navigation through all interactive elements
- Manual check: screen reader announcement of carousel controls and newsletter feedback

### Test Tag Format

All property-based tests carry a comment in the format:
```
// Feature: streetwear-ootd-website, Property N: <property_text>
```
