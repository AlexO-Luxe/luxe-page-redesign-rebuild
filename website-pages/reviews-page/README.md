# Chorus reviews page — section blocks

Self-hosted rebuild of `/our-reviews`. Replaces the Elfsight widget (the iOS
Safari crash source). Split into five **self-contained** Code Blocks: each block
carries its own CSS and JS, exactly like `atlas-cities-page.html`. Nothing goes
in Code Injection — paste each block into its own section, in order.

## Paste order (all on the `/our-reviews` page, top to bottom)

1. `01-featured-guests.html` — header intro + guest tiles + modal (the focal section)
2. `02-trust-stats.html` — rating / reviews / since / cities strip
3. `03-social-videos.html` — TikTok + Instagram video wall
4. `04-written-reviews.html` — verified Google review wall
5. `05-cta-band.html` — closing enquiry CTA

Each is one **Code Block**. Set each Squarespace section background to
transparent/light; the CTA and video/guest cards bring their own dark surfaces.
Blocks are independent — reorder or drop one without touching the others.

## What to edit (all inside the `<script>` CONFIG at the top of each block)

- **01 – guests:** per guest set `image` (Squarespace CDN photo), `video`
  (YouTube link *or* a direct `.mp4` url; add `portrait:true` for vertical),
  `quote`, and confirm the `href` slug (`/guest-reviews/...`). Blank `video` =
  text-only modal; blank `image` = branded initial card.
- **02 – stats:** the four numbers + `googleReviewsUrl`.
- **03 – videos:** `platform`, public `url`, `poster` screenshot, `city`, `line`.
- **04 – reviews:** `name`, `initials`, `quote`, Google `url`. Shows 6 then "show more".
- **05 – CTA:** the two `href=""` values (enquiry page + Google review link).

## iOS-Safari safety (per `docs/ios-safari-scroll-rules.md`)

- No third-party embed loads until a guest taps play. Guest modal and social
  cards each hold at most one iframe/video at a time and destroy it on close, so
  playback always stops and compositor memory stays low.
- No `backdrop-filter` blur on the fixed modal scrim (opaque instead).
- Reveal is opacity-only via IntersectionObserver — no scroll handlers, no
  persistent `will-change`/`translateZ` on the repeated tiles.
- Guest photos + video posters are `loading="lazy" decoding="async"` at `?format=750w`.

## Progressive enhancement / SEO

Each guest tile is a real `<a href="/guest-reviews/...">`; the modal is a
`preventDefault` enhancement. No-JS visitors and crawlers still reach the guest
pages.

## Note

`../chorus-reviews-page.html` is the earlier single-block version. These split
blocks supersede it; delete the monolith once you have pasted these.
