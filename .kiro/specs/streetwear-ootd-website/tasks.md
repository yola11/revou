# Implementation Plan: Streetwear OOTD Website

## Overview

Implement a single `index.html` file delivering a premium streetwear OOTD editorial experience. All styling is provided by Tailwind CSS (Play CDN) with supplemental custom CSS, and all interactivity is implemented in embedded Vanilla JavaScript (ES2020). The implementation follows the architecture defined in design.md, building from the base skeleton outward — scaffolding, then global systems, then section by section, finishing with QA passes.

---

## Tasks

- [x] 1. Base HTML skeleton, Tailwind CDN, and config block
  - Create `index.html` with `<!DOCTYPE html>`, `<html lang="en">`, `<head>`, and `<body>` scaffolding
  - Add `<meta charset="UTF-8">` and `<meta name="viewport" content="width=device-width, initial-scale=1.0">`
  - Add `<title>` and Tailwind Play CDN `<script src="https://cdn.tailwindcss.com">`
  - Add the inline `tailwind.config` `<script>` block with custom colors (`bg-base`, `bg-surface`, `off-white`, `accent-blue`, `accent-purple`), `fontFamily.display`, custom `animation` names, and all `keyframes` (`gradientShift`, `fadeInOverlay`, `pulseBounce`, `badgePulse`)
  - Add empty `<style>` block placeholder after the config script
  - Add `#page-transition`, `#glow-cursor`, `<header id="navbar">`, `<main>` with all eight `<section>` stubs (ids: `hero`, `featured`, `trending`, `new-arrivals`, `gallery`, `story`, `reviews`, `newsletter`), and `<footer id="footer">`
  - Add empty `<script>` block at end of `<body>` for JS modules
  - _Requirements: 1.1, 1.2, 1.6_

- [x] 2. CSS custom properties, global resets, and animation utility classes
  - Write `:root` block with all palette variables (`--color-bg`, `--color-surface`, `--color-text`, `--color-text-muted`, `--color-accent-blue`, `--color-accent-purple`), spacing tokens, typography tokens, and animation tokens as specified in design.md
  - Write `body` reset: `background-color: var(--color-bg); color: var(--color-text); font-family: var(--font-display)`
  - Write `.glass` utility class with `background`, `backdrop-filter`, `-webkit-backdrop-filter`, and `border`
  - Write `[data-animate]` initial hidden state (`opacity: 0; transform: translateY(30px); transition: opacity 700ms ease-out, transform 700ms ease-out`) and `.is-visible` revealed state
  - Write `@media (prefers-reduced-motion: reduce)` block disabling all `[data-animate]` transitions, `.scroll-indicator` animation, and `#page-transition` animation
  - Write `html { scroll-behavior: smooth }` global smooth scroll
  - Write `:focus-visible` outline rule (2px solid `var(--color-accent-blue)`, 2px offset) for all interactive elements
  - _Requirements: 1.3, 1.5, 1.6, 14.2, 14.4, 14.5_

- [x] 3. Page Transition overlay
  - Add CSS for `#page-transition`: `position: fixed; inset: 0; background: var(--color-bg); z-index: 100; animation: fadeInOverlay 1.2s ease forwards`
  - Implement `PageTransition` IIFE in the JS block: call `init()` at module level, attach `animationend` listener that sets `element.style.display = 'none'`
  - Apply `prefers-reduced-motion` guard: if reduced motion, immediately hide the overlay without animation
  - _Requirements: 1.4_

- [~] 4. Mouse-Follow Glow element and MouseGlow JS module
  - Add CSS for `#glow-cursor` (position fixed, 600×600px, border-radius 50%, pointer-events none, transform translate(-50%,-50%), radial gradient `rgba(59,130,246,0.10)` → transparent, transition `left 100ms linear, top 100ms linear`, z-index 0)
  - Implement `MouseGlow` IIFE: check `window.innerWidth < 768` → return early; attach `mousemove` listener on `document`; on move set `glowEl.style.left` and `glowEl.style.top` to `e.clientX` / `e.clientY`
  - Apply `prefers-reduced-motion` JS guard: skip init if reduced motion is active
  - _Requirements: 12.1, 12.2, 12.3, 12.4_

  - [ ]* 4.1 Write property test for Property 4 (mouse glow position tracks cursor) and Property 5 (glow disabled on mobile)
    - Set up Vitest + fast-check in a `test/` directory alongside `index.html`; create `src/utils.js` exporting `computeGlowPosition` and `isMobileViewport` pure functions extracted from `MouseGlow`
    - **Property 4:** For any `(x, y)` on desktop viewport (width ≥ 768px), `computeGlowPosition(x, y)` returns `{ left: x, top: y }`
    - **Property 5:** For any viewport width < 768px, `isMobileViewport(width)` returns `true` and no glow position is computed
    - **Validates: Requirements 12.1, 12.3, 12.4**

- [~] 5. Glassmorphism Navbar
  - Write Navbar HTML inside `<header id="navbar">`: brand wordmark `<a>`, `<nav>` with desktop links (`<a>` anchors to each section), and hamburger `<button aria-label="Toggle menu">` with `aria-expanded`
  - Write mobile overlay menu HTML: full-screen `<div class="mobile-menu">` containing `<nav>` with link list and close button; apply focus trap loop in JS
  - Add CSS: `header { position: fixed; top: 0; width: 100%; z-index: 50; transition: ... }`, `.scrolled` selector applying `.glass` styles and `border-bottom`
  - Add mobile overlay CSS: `position: fixed; inset: 0; z-index: 60; transform: translateX(100%); transition: transform 300ms ease`; `.menu-open` sets `translateX(0)`
  - Add stagger CSS for mobile nav links: nth-child delay increments using inline style set by JS
  - Implement `NavbarController` IIFE: `init()` attaches passive scroll listener calling `handleScroll`; attaches click listeners to all nav `<a>` elements calling `handleNavClick`; attaches click to hamburger calling `toggleMobileMenu`
  - `handleScroll(scrollY)`: `scrollY > 50` → `classList.add('scrolled')`; else `classList.remove('scrolled')`
  - `handleNavClick(anchor)`: `document.querySelector(anchor).scrollIntoView({ behavior: 'smooth' })`
  - `toggleMobileMenu()`: toggle `menu-open` class on overlay; set stagger `animationDelay` on each link child; manage `aria-expanded`; activate/deactivate focus trap
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6_

  - [ ]* 5.1 Write property test for Property 6 (navbar scroll state) and Property 7 (nav link scroll target)
    - Export `handleScroll(scrollY)` → `boolean` (true = scrolled class should be present) and `resolveAnchor(href)` → `string` as pure functions in `src/utils.js`
    - **Property 6:** For any `scrollY > 50`, function returns `true`; for any `scrollY ≤ 50`, returns `false`
    - **Property 7:** For any anchor string `#<id>`, `resolveAnchor` returns the same `#<id>` unchanged (smoke test; DOM integration tested separately)
    - **Validates: Requirements 2.2, 2.4**

- [x] 6. Hero Banner section
  - Write Hero HTML inside `<section id="hero" aria-labelledby="hero-heading">`: two-layer structure — `<div id="hero-bg" aria-hidden="true">` (absolute, full coverage, AnimatedGradient background, `will-change: transform`) and `<div id="hero-fg">` (relative z-1, flex column center)
  - Inside `#hero-fg`: oversized `<h1 id="hero-heading" data-animate>` headline, `<p data-animate>` sub-copy, `<button data-magnetic data-animate class="cta-primary">` CTA, `<div class="scroll-indicator" aria-hidden="true">` with bounce SVG arrow
  - Add CSS: `#hero { min-height: 100vh; position: relative; overflow: hidden }`, `#hero-bg` animated gradient cycling accent blue → accent purple at ~4% opacity, `#hero-fg` flex layout centered, `h1` clamp font-size, scroll indicator `animate-pulse-bounce`
  - Stagger `data-animate` children with initial CSS hidden state; `ScrollAnimationObserver` (task 17) will reveal them
  - _Requirements: 3.1, 3.2, 3.4, 3.5, 3.6, 3.7_

- [x] 7. JavaScript data arrays
  - Declare `FEATURED_PRODUCTS` array (min 4 items) with fields `id`, `name`, `category`, `price`, `imageSrc` (Unsplash URLs with `?w=800&q=75`), `imageAlt`, `isNew`, `size` (`large`/`medium`/`small`)
  - Declare `NEW_ARRIVALS` array (min 4 items) same shape; at least one item has `isNew: true`
  - Declare `OOTD_CARDS` array (min 3 items) with fields `id`, `title`, `descriptor`, `imageSrc`, `imageAlt`, `shopLabel`
  - Declare `GALLERY_IMAGES` array (min 6 items) with fields `id`, `imageSrc`, `imageAlt`, `caption`, `span` (`tall`/`wide`/`square`)
  - Declare `REVIEWS` array (min 3 items) with fields `id`, `name`, `rating` (1–5 integer), `text`, `location`
  - Declare `AGGREGATE_RATING` object with `score` (float) and `count` (integer)
  - _Requirements: 4.2, 5.1, 6.2, 7.2, 9.1, 9.4_

- [~] 8. Featured Collection section
  - Write section HTML: `<section id="featured" aria-labelledby="featured-heading" data-animate data-stagger-parent>` with eyebrow text element and `<h2 id="featured-heading">`, then `<div id="featured-grid">` (JS render target)
  - Add CSS: `#featured-grid { display: grid; grid-template-columns: 2fr 1fr 1fr }` on desktop; first item `grid-row: span 2`; on `< 768px` reflow to `grid-template-columns: 1fr 1fr`
  - Implement `renderFeaturedCollection()` function: iterates `FEATURED_PRODUCTS`, creates `<article data-animate>` card per product with `<img loading="lazy">`, `<h3>` name, `<p>` category/price, hover zoom via CSS class; appends to `#featured-grid`
  - Add hover zoom CSS: `article.product-card img { transition: transform 400ms ease }` + `.product-card:hover img { transform: scale(1.08) }`
  - Call `renderFeaturedCollection()` in `DOMContentLoaded` handler
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6_

  - [ ]* 8.1 Write property test for Property 13 (content data completeness — product cards)
    - Export `buildProductCard(product)` → DOM string/node from `src/utils.js`; run with Vitest + JSDOM
    - **Property 13 (products):** For any valid `Product` object, the rendered card contains an `<img>`, a name text node, and a price/category label
    - **Validates: Requirements 4.2**

- [~] 9. Trending OOTD section
  - Write section HTML: `<section id="trending" aria-labelledby="trending-heading">` with oversized `<h2 id="trending-heading">` (negative margin-bottom to overlap first card), `<div id="ootd-grid" data-stagger-parent>` (JS render target)
  - Add CSS: desktop masonry grid (`grid-template-columns: repeat(3, 1fr); grid-auto-rows: ...`) with one card `grid-row: span 2`; mobile horizontal scroll `display: flex; overflow-x: auto; scroll-snap-type: x mandatory` with cards `scroll-snap-align: start; min-width: 80vw`
  - Implement `renderOOTDCards()` function: iterates `OOTD_CARDS`, creates `<article data-animate>` with `<img loading="lazy">`, `<h3>` title, `<p>` descriptor, and absolutely-positioned overlay div (opacity 0, reveals on hover with "Shop the Look" label)
  - Add hover overlay CSS: `.ootd-card .overlay { position: absolute; inset: 0; opacity: 0; transition: opacity 300ms ease; background: rgba(0,0,0,0.55) }` + `.ootd-card:hover .overlay { opacity: 1 }`
  - Call `renderOOTDCards()` in `DOMContentLoaded`
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

  - [ ]* 9.1 Write property test for Property 13 (content data completeness — OOTD cards)
    - Export `buildOOTDCard(card)` from `src/utils.js`
    - **Property 13 (OOTD):** For any valid `OOTDCard`, rendered card contains `<img>`, title element, and descriptor element
    - **Validates: Requirements 5.1**

- [~] 10. New Arrivals section
  - Write section HTML: `<section id="new-arrivals" aria-labelledby="arrivals-heading">` with `<h2 id="arrivals-heading">`, `<div id="arrivals-grid" data-stagger-parent>` (JS render target), and `<button data-magnetic>` "Explore Drop" CTA
  - Add CSS: `#arrivals-grid { display: grid; grid-template-columns: 2fr 1fr 1fr }` with first item `grid-row: span 2`; hover zoom `.arrivals-card:hover img { transform: scale(1.06) }` over 350ms
  - Add "NEW DROP" badge CSS: `position: absolute; top: 1rem; left: 1rem; animation: badgePulse 1.5s ease-in-out infinite; border: 1px solid currentColor; padding: 0.25rem 0.5rem`
  - Implement `renderNewArrivals()` function: iterates `NEW_ARRIVALS`, creates `<article data-animate>` cards; if `product.isNew` is true, injects the "NEW DROP" badge element
  - Call `renderNewArrivals()` in `DOMContentLoaded`
  - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [~] 11. Style Inspiration Gallery section
  - Write section HTML: `<section id="gallery" aria-labelledby="gallery-heading">` with eyebrow text, `<h2 id="gallery-heading">`, `<div id="gallery-mosaic" data-stagger-parent>` (JS render target)
  - Add desktop mosaic CSS: `display: grid; grid-template-columns: repeat(3, 1fr); grid-auto-rows: 300px`; items with `span: 'tall'` get `grid-row: span 2`; items with `span: 'wide'` get `grid-column: span 2`
  - Add mobile CSS `< 640px`: `grid-template-columns: 1fr 1fr; grid-auto-rows: 240px` (uniform two-column)
  - Implement `renderGallery()` function: iterates `GALLERY_IMAGES`, creates `<figure data-animate>` with `<img loading="lazy">` and absolutely-positioned overlay `<figcaption>` (opacity 0, transitions to 1 on hover + scale 1.06 on img)
  - Call `renderGallery()` in `DOMContentLoaded`
  - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 7.6_

- [~] 12. Brand Story section
  - Write section HTML: `<section id="story" aria-labelledby="story-heading">` with dark overlay wrapper, manifesto `<h2 id="story-heading" data-animate>`, three `<p data-animate>` body copy paragraphs, and `<blockquote data-animate>` accent element with oversized `"` glyph
  - Add CSS: `#story { width: 100%; min-height: 80vh; position: relative; overflow: hidden }` with AnimatedGradient background cycling accent colors; `#story-overlay { position: absolute; inset: 0; background: rgba(10,10,10,0.65) }`; `h2` clamp font-size `clamp(2.5rem, 6vw, 5rem)`
  - Ensure text contrast: off-white `#f5f5f0` on `rgba(10,10,10,0.65)` overlay ≥ 4.5:1 (confirmed in design.md)
  - Add 200ms stagger between `<h2>` and subsequent `<p>` elements via inline `animation-delay` or `[data-animate]` stagger computed by `ScrollAnimationObserver`
  - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_

- [~] 13. Checkpoint — Ensure all sections and global styles render correctly
  - Ensure all tests pass, ask the user if questions arise.

- [~] 14. Customer Reviews section and CarouselController
  - Write section HTML: `<section id="reviews" aria-labelledby="reviews-heading">` with aggregate rating row (`<div>` showing `★ {score} / 5 · {count} reviews`), `<div id="reviews-carousel">` (render target), and prev/next `<button aria-label="Previous review">` / `<button aria-label="Next review">` arrows plus dots container `<div id="carousel-dots">`
  - Add CSS: desktop `#reviews-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1.5rem }`; mobile single-card carousel with `.slide-track { display: flex; transition: transform 400ms ease }`; review cards use `.glass` class + `border-radius: 1rem; padding: 1.5rem`
  - Implement `CarouselController` IIFE: `init(carouselEl, reviewData)` renders review `<article>` cards into slide track, attaches prev/next listeners calling `goTo`, if `reviewData.length > 3` renders dots via `renderDots()`
  - `goTo(index)`: clamp `index` to `[0, reviewData.length - 1]`; set `slideTrack.style.transform = \`translateX(\${-index * 100}%)\``; update active dot; disable prev button at 0, next button at max
  - `renderDots()`: create N dot `<button>` elements; active dot gets distinct style; clicking dot calls `goTo(i)`
  - Implement `renderReviews()` function that passes `REVIEWS` to `CarouselController.init()`; call in `DOMContentLoaded`
  - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5, 9.6_

  - [ ]* 14.1 Write property test for Property 13 (content data completeness — review cards)
    - Export `buildReviewCard(review)` from `src/utils.js`
    - **Property 13 (reviews):** For any valid `Review` (rating integer 1–5), rendered card contains name, star rating element, and review text
    - **Validates: Requirements 9.1**

  - [ ]* 14.2 Write unit tests for CarouselController bounds clamping
    - Test `goTo(-1)` → stays at 0; `goTo(reviewData.length)` → stays at `reviewData.length - 1`; `goTo(1)` → moves to index 1
    - Test dot count equals `reviewData.length`
    - _Requirements: 9.5_

- [~] 15. Newsletter section and NewsletterForm module
  - Write section HTML: `<section id="newsletter" aria-labelledby="newsletter-heading">` with `<h2 id="newsletter-heading">`, value proposition `<p>`, `<form id="newsletter-form">` containing `<input type="email" aria-label="Email address">`, submit `<button data-magnetic>`, inline `<p id="newsletter-error" role="alert" aria-live="polite">` error element, and `<div id="newsletter-success">` success message div
  - Add CSS: AnimatedGradient background (`background-size: 200% 200%; animation: gradientShift 8s ease infinite`) cycling accent blue → purple at ~15% opacity on dark base; form `display: flex; gap: 0.5rem` desktop, stacked mobile; success/error transitions via `opacity` and `max-height`
  - Implement `NewsletterForm` IIFE: `init()` attaches `submit` listener; `validate(email)` uses regex `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`; `handleSubmit(e)` calls `e.preventDefault()`, branches on `validate`; `showSuccess()` hides form, fades in success div over 300ms; `showError(msg)` shows error element with message
  - Extract `validateEmail(str)` → `boolean` as a pure function in `src/utils.js` (shared with test)
  - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5, 10.6_

  - [ ]* 15.1 Write property test for Property 8 (newsletter email validation state)
    - **Property 8:** For any string matching `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`, `validateEmail` returns `true`; for any string not matching (including empty string), returns `false`
    - Use fast-check `fc.emailAddress()` arbitraries and invalid string arbitraries
    - **Validates: Requirements 10.3, 10.4**

- [~] 16. Premium Footer
  - Write footer HTML: `<footer id="footer" aria-label="Site footer">` with top border `<hr aria-hidden="true">`, four-column grid: brand column (wordmark, tagline, copyright), three link columns (Shop, About, Support each as `<nav aria-label="...">` with `<ul>` links), social icons row, and back-to-top `<button aria-label="Back to top">`
  - Add CSS: `#footer { display: grid; grid-template-columns: 2fr 1fr 1fr 1fr; gap: 2rem; border-top: 1px solid rgba(255,255,255,0.08) }`; mobile `< 640px` reflow to `grid-template-columns: 1fr` with `<details>/<summary>` accordion on link columns
  - Social icon hover: `transition: opacity 200ms ease, color 200ms ease`
  - Back-to-top click handler: `window.scrollTo({ top: 0, behavior: 'smooth' })`
  - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5, 11.6_

- [~] 17. ScrollAnimationObserver JS module
  - Implement `ScrollAnimationObserver` IIFE in JS block
  - `init()`: check `prefers-reduced-motion` — if true, immediately add `is-visible` to all `[data-animate]` elements and return; else create `IntersectionObserver(callback, { threshold: 0.15 })`; `querySelectorAll('[data-animate]')` → observe each; if `IntersectionObserver` not in window, fallback synchronous loop adding `is-visible`
  - Callback: for each intersecting entry, `entry.target.classList.add('is-visible')`; if target has `[data-stagger-parent]`, iterate direct animated children setting `animationDelay = \`${i * 120}ms\``; call `observer.unobserve(entry.target)`
  - After reveal, attach `transitionend` listener on each element to remove `will-change: transform` style
  - Export `computeStaggerDelay(index, baseMs)` → `number` as pure function in `src/utils.js`
  - _Requirements: 14.1, 14.2, 14.3, 14.4, 14.5_

  - [ ]* 17.1 Write property test for Property 1 (scroll-triggered class application and staggered delay)
    - **Property 1:** For any `index ≥ 0` and `baseMs = 120`, `computeStaggerDelay(index, 120)` returns `index * 120`; consecutive pairs differ by exactly `baseMs` (80–150ms range check)
    - **Property 11:** For any N consecutive siblings (N ≥ 2), each consecutive pair's delay difference is between 80ms and 150ms inclusive
    - **Validates: Requirements 4.4, 5.4, 6.4, 7.4, 8.3, 9.3, 14.1, 14.3, 14.4**

- [~] 18. ParallaxEngine JS module
  - Implement `ParallaxEngine` IIFE: `init()` checks `prefers-reduced-motion` — if true, return; checks `window.innerWidth` (no strict mobile gate, parallax is scroll-based); attaches `scroll` listener `{ passive: true }` on `window`
  - `handleScroll(scrollY)`: compute `bgOffset = scrollY * 0.5`; apply `heroBackground.style.transform = \`translateY(${bgOffset}px)\``
  - Export `computeParallaxOffset(scrollY, factor)` → `number` as pure function in `src/utils.js` (`factor = 0.5`)
  - _Requirements: 3.3_

  - [ ]* 18.1 Write property test for Property 12 (parallax background moves at 40–60% of scroll speed)
    - **Property 12:** For any `scrollY ≥ 0`, `computeParallaxOffset(scrollY, 0.5)` returns a value between `0.40 * scrollY` and `0.60 * scrollY` inclusive
    - **Validates: Requirements 3.3**

- [~] 19. MagneticButton JS module
  - Implement `MagneticButton` IIFE: `init()` checks `window.innerWidth < 768` → return; checks `prefers-reduced-motion` → return; `querySelectorAll('[data-magnetic]')` → attach `mousemove` and `mouseleave` listeners to each
  - `handleMouseMove(button, e)`: get `rect = button.getBoundingClientRect()`; if `rect.width === 0` return early; compute `cx`, `cy`, `dx`, `dy`, `dist = Math.hypot(dx, dy)`; `PROXIMITY_RADIUS = 50`; if `dist < PROXIMITY_RADIUS`: `SHIFT = 0.275`; set `tx = dx * SHIFT`, `ty = dy * SHIFT`; apply via rAF guard (set rafPending flag, call `requestAnimationFrame`, clear flag in callback, skip if already pending)
  - `handleMouseLeave(button)`: `button.style.transform = 'translate(0, 0)'; button.style.transition = 'transform 350ms ease-out'`
  - Export `computeMagneticShift(cx, cy, ex, ey, proximityRadius, shiftFactor)` → `{ tx, ty }` as pure function in `src/utils.js`
  - _Requirements: 13.1, 13.2, 13.3, 13.4_

  - [ ]* 19.1 Write property test for Property 2 (magnetic button shift within proximity)
    - **Property 2:** For any cursor position where `dist < proximityRadius`, the resulting shift ratio `Math.hypot(tx, ty) / dist` is between 0.20 and 0.35
    - Use fast-check to generate `(cx, cy, ex, ey)` filtered so `Math.hypot(ex-cx, ey-cy) < 50` and `> 0`
    - **Validates: Requirements 13.1**

  - [ ]* 19.2 Write property test for Property 3 (magnetic button return on departure)
    - **Property 3:** For any shifted position, `handleMouseLeave` resets transform to `translate(0, 0)` with `transition: transform 350ms ease-out`; verify transition duration is ≤ 400ms
    - Export `computeReturnTransition()` → `{ transform: string, transition: string }` from `src/utils.js`
    - **Validates: Requirements 13.2**

- [~] 20. Image error handling (onerror placeholder)
  - Add `onerror` attribute to every `<img>` element in the document (both static and render-function generated)
  - The `onerror` handler sets `this.src` to an inline dark SVG placeholder: `data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' width='800' height='600'><rect width='800' height='600' fill='%231a1a1a'/></svg>`
  - For render-function-generated images, set the `onerror` attribute as part of element creation in each `render*` function
  - _Requirements: 15.5 (img integrity)_

- [~] 21. Accessibility wiring
  - Verify all `<section>` elements have `aria-labelledby` pointing to their `<h2>` id
  - Verify all `<article>` product/review cards have descriptive content (role is implicit from element)
  - Add `role="alert"` and `aria-live="polite"` to `#newsletter-error` and `#newsletter-success`
  - Add `aria-label` to all icon-only buttons (hamburger, close, carousel prev/next, back-to-top, social icons)
  - Add `aria-label` with `"{N} out of 5 stars"` to all star rating visual elements in review cards
  - Add `aria-hidden="true"` to purely decorative elements (`#glow-cursor`, `#hero-bg`, overlay divs, `#page-transition`)
  - Implement focus trap for mobile menu overlay: on `menu-open`, collect all focusable children; on Tab/Shift+Tab at boundary, wrap focus; on Escape, close menu
  - Add `role="presentation"` or `aria-hidden="true"` to decorative `<img>` background elements
  - _Requirements: 2.6, 8.5, 9.6, 10.3, 10.4, 15.5, 15.6_

  - [ ]* 21.1 Write property test for Property 10 (all images have non-empty alt attributes)
    - Parse `index.html` with JSDOM in Vitest; query all `<img>` elements
    - **Property 10:** Every `<img>` has `alt` attribute that is non-empty (`alt !== ''` and `alt !== null`)
    - **Validates: Requirements 15.5**

- [~] 22. Responsive QA pass
  - Open `index.html` at 320px, 768px, 1024px, and 1440px viewport widths in browser devtools
  - Verify no unintended horizontal scroll (`document.body.scrollWidth <= window.innerWidth`) at each breakpoint
  - Verify Featured_Collection reflows to two-column on `< 768px`
  - Verify Trending_OOTD shows horizontal scroll with snap on mobile, masonry grid on desktop
  - Verify Gallery reflows to two-column uniform on `< 640px`
  - Verify Footer stacks to single-column on `< 640px`
  - Fix any layout overflow, overlap, or spacing issues found at each breakpoint
  - Write a Vitest structural test using JSDOM that loads `index.html` and checks `document.querySelectorAll('[data-animate]').length > 0` and absence of inline `opacity: 0` styles after init
  - _Requirements: 15.1, 15.4_

  - [ ]* 22.1 Write property test for Property 9 (all non-hero images carry lazy loading attribute)
    - Parse `index.html` with JSDOM; query all `<img>` elements not inside `#hero`
    - **Property 9:** Every such `<img>` has `loading="lazy"` attribute
    - **Validates: Requirements 15.2**

- [~] 23. Performance QA pass
  - Verify all `<img>` outside `#hero` have `loading="lazy"` attribute in both static HTML and render functions
  - Add `fetchpriority="high"` to the hero background image (if an `<img>` is used) or the hero foreground `<img>`
  - Add `width` and `height` attributes to all `<img>` elements to reserve layout space and prevent CLS
  - Verify `will-change: transform` is only applied to `#hero-bg`, `#glow-cursor`, and actively animating `[data-animate]` elements (removed on `transitionend`)
  - Verify all scroll event listeners use `{ passive: true }` flag
  - Verify MagneticButton mousemove uses rAF guard (review code, ensure `rafPending` flag prevents queuing)
  - Add Unsplash URL width params (`?w=800&q=75`) to all image URLs in the data arrays if not already present
  - _Requirements: 15.2, 15.3_

- [~] 24. Final checkpoint — Ensure all tests pass
  - Run `vitest --run` in the project root and confirm all property-based and unit tests pass
  - Ensure all tests pass, ask the user if questions arise.

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP — they are property-based or unit tests validating correctness properties from design.md
- Each task references specific requirements for traceability
- Checkpoints (tasks 13 and 24) ensure incremental validation at natural break points
- All property-based tests use fast-check with `numRuns: 100`; run with `vitest --run`
- Pure functions (`computeMagneticShift`, `computeParallaxOffset`, `validateEmail`, `computeStaggerDelay`, `computeGlowPosition`, `buildProductCard`, `buildOOTDCard`, `buildReviewCard`) are extracted into `src/utils.js` for isolated testing
- The single `index.html` output file embeds all CSS and JS — the `src/utils.js` and `test/` files are development artifacts only and are not served
- `IntersectionObserver` feature check gates the observer — fallback immediately reveals all `[data-animate]` elements
- `prefers-reduced-motion` guards are applied in both CSS (`@media` block) and JS (before attaching parallax, magnetic, and page-transition listeners)

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1"] },
    { "id": 1, "tasks": ["2", "7"] },
    { "id": 2, "tasks": ["3", "4", "5", "6"] },
    { "id": 3, "tasks": ["4.1", "5.1", "8", "9", "10", "11", "12"] },
    { "id": 4, "tasks": ["8.1", "9.1", "14", "15", "16", "17", "18", "19"] },
    { "id": 5, "tasks": ["14.1", "14.2", "15.1", "17.1", "18.1", "19.1", "19.2", "20"] },
    { "id": 6, "tasks": ["21"] },
    { "id": 7, "tasks": ["21.1", "22"] },
    { "id": 8, "tasks": ["22.1", "23"] }
  ]
}
```
