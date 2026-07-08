# luxe-page-redesign-rebuild — Student Luxe website rebuild

Squarespace builds for studentluxe.co.uk: header/nav, page redesigns, and the
design system behind them. Files here are pasted into Squarespace **Code
Injection** (site-wide) or **Code Blocks** (per page).

## Repo layout
- `header-nav/aperture-squarespace-header.html` — the site header/nav ("Aperture").
  Goes in Settings → Advanced → Code Injection → **Header**.
- `pages/` — one self-contained file per page build (e.g. `atlas-cities-page.html`).
  Each goes in a **Code Block** on its page.
- `docs/design-system.md` — palette, type, motion, component patterns.
- `docs/ios-safari-scroll-rules.md` — **mandatory** crash-avoidance rules.

## Non-negotiable rules
1. **Read `docs/ios-safari-scroll-rules.md` before writing any HTML/CSS/JS.**
   Never: backdrop-filter blur on sticky/fixed elements; persistent
   `will-change`/`translateZ(0)` on repeated cards; eager full-res CSS
   background images on long lists. Always: `<img loading="lazy"
   decoding="async">` with right-sized `?format=` (≈750w tiles);
   IntersectionObserver instead of scroll handlers; honour
   `prefers-reduced-motion`.
2. **Never animate `transform` on tap targets while they are tappable** —
   iOS cancels taps on moving targets (caused a two-tap bug once already).
   Reveal with opacity only.
3. **Namespace everything** (`#sl-ap`, `.atlas`, …) — no bare element selectors
   that could leak into the Squarespace theme.
4. **Config-first**: all links, cities, labels live in one CONFIG object at the
   top of each file. Editors should never touch markup.
5. **Inputs ≥16px font-size on mobile** — smaller triggers iOS focus-zoom.
6. **Complete files only** — deliverables are whole paste-ready files, not diffs.

## Design system (short form — full version in docs/design-system.md)
- Palette: paper `#FBF8F1` / ivory `#F4F0E6`; ink `#0d1a2e`; gold `#B8966E`
  (hover `#a07d5a`, deep `#8B6E4E`); alert red `#C53026`.
- Type (Adobe Typekit, already loaded site-wide): `baskerville-display-pt`
  display serif (tight tracking, italic gold accent); `dm-sans` UI (uppercase,
  wide tracking); `adobe-garamond-pro` body serif.
- Motion: ease `cubic-bezier(.22,.8,.28,1)`; FLIP for grid re-layout; staggered
  opacity reveals; hover = image push-in + gold hairline + arrow.
- Buttons: 8px-radius rectangles, DM Sans 11px `.12em` uppercase, icon-left;
  gold = primary (matches site's "Make an enquiry"), ink/navy = secondary.
  Floating buttons sit at `bottom:24px` (16px below 400px width) to align with
  the site's enquiry widget.

## Workflow expectations
- Validate JS with `node --check` on the extracted `<script>` before delivering.
- Render-test desktop (1440px) and mobile (390–402px, `isMobile:true`) with the
  pre-installed Playwright Chromium; screenshot-verify changes before shipping.
- Note: the sandbox has no network — Squarespace CDN images show as fallbacks
  in screenshots; that's expected.
- The live site's tracking stack (Consent Mode, GTM, Cookiebot, UTM capture)
  lives in the Squarespace injections, NOT in this repo — never bundle or
  modify tracking here.

## Related
- Main marketing-dashboard repo: `AlexO-Luxe/luxe-organic-content`
  (Next.js dashboard; the `redesign/` folder there was the original home of
  these files during development).
