# iOS / Safari scroll-crash rules (ALWAYS follow)

Any HTML/CSS/JS built for the marketing site (code injections, code blocks) must
avoid the Safari/iOS **"unexpected error"** crash triggered by fast scrolling.
On real iPhones this reloads the tab; it is caused by **GPU-layer / decoded-image
memory exhaustion**, not a script error.

## Rules

1. **No `backdrop-filter` / `-webkit-backdrop-filter` blur on `position:sticky`
   or `position:fixed` elements.** Blur on a pinned element is the single
   biggest trigger. Use an opaque / near-opaque background instead.

2. **No persistent `will-change` or `transform:translateZ(0)` / `translate3d`
   on elements that repeat** (cards, tiles, list items). Each one becomes its
   own compositing layer; 40+ of them exhausts the compositor. Promote to a
   layer only *transiently* during an animation, then clear the inline style.

3. **Lazy-load images at scale.** Once a list can grow past ~10 items, use
   `<img loading="lazy" decoding="async">` with a right-sized Squarespace
   `?format=` (≈`750w` for a tile) — **not** eager full-res CSS
   `background-image`. Only images near the viewport should ever decode.

4. **Keep simultaneous composited layers low.** Prefer opacity/transform
   transitions that start and end cleanly. Don't leave elements in a
   permanently-promoted state.

5. **Avoid heavy per-frame scroll handlers.** Use `IntersectionObserver` for
   scroll-triggered UI (reveal-on-scroll, "back to top" buttons) instead of a
   `scroll` listener that reads layout every frame.

6. **Honour `prefers-reduced-motion`.** Disable non-essential transforms/motion
   under the reduced-motion media query.

## Quick audit checklist

- [ ] No blur filters on sticky/fixed elements
- [ ] No blanket `will-change` / `translateZ(0)` on repeated cards
- [ ] Images lazy-loaded + right-sized once the list is long
- [ ] Scroll-driven UI uses `IntersectionObserver`, not a `scroll` handler
- [ ] `prefers-reduced-motion` fallback present

Both builds in this repo (`aperture-squarespace-header.html`,
`atlas-cities-page.html`) already comply — use them as the reference pattern.
