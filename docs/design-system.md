# Luxe design system

The shared visual language behind every build in this repo. Values are the
source of truth; each file re-declares them as CSS custom properties scoped to
its own namespace so nothing leaks into the Squarespace theme.

## Palette

| Token | Value | Use |
|-------|-------|-----|
| Paper (ivory) | `#FBF8F1` | Page / surface background |
| Ivory deep | `#F4F0E6` | Secondary surface |
| Ink (navy) | `#0d1a2e` (`#0a0a0a` in header) | Primary text, dark pills |
| Muted ink | `rgba(13,26,46,.52)` | Secondary text |
| Hairline | `rgba(13,26,46,.12)` | Borders, rules |
| Gold / bronze | `#B8966E` | Primary accent |
| Gold hover/deep | `#a07d5a` / `#8B6E4E` | Accent hover, small labels |
| Gold wash | `rgba(184,150,110,.10–.16)` | Tints, chips |
| Alert red | `#C53026` | Destructive affordance (e.g. clear-search ✕) |

**Dark variant:** both files are token-driven. Swap the paper/ink pair
(paper → navy, ink → cream) and the whole surface inverts — see the
"DARK FLIP" comment block in `atlas-cities-page.html`.

## Type

Loaded site-wide via Adobe Typekit (already on the Squarespace site):

- **Display / headings:** `baskerville-display-pt`, Georgia, serif — used at
  large sizes with tight tracking (`letter-spacing:-.03em`) and an *italic*
  bronze accent for the second line of a title.
- **UI / labels:** `dm-sans`, system-ui — uppercase, wide tracking
  (`.12–.18em`) for eyebrows, pills, nav.
- **Body serif:** `adobe-garamond-pro`, Georgia — running copy, quotes, blurbs.

## Motion

- Standard easing: `cubic-bezier(.22,.8,.28,1)`.
- Reveals: opacity + short `translateY`, staggered on first paint.
- Filtering: **FLIP** (measure → re-render → invert → play) so tiles glide to
  new positions rather than hard-swapping.
- Hover: image push-in (`scale(1.06)`), gold hairline ignite, arrow draw-in,
  label lift. Layers promoted only during the transition (see iOS rules).

## Components / patterns

- **City tile:** photo (lazy `<img>`), bottom gradient, country eyebrow +
  serif name, optional gold tag, hover arrow. Uniform aspect-ratio for scanning.
- **Pills / chips:** rounded (`border-radius:100px`), gold-wash idle, ink when
  active.
- **Control bar:** search + autocomplete, region tabs with live counts, sort.
- **Return-to-filter pill:** fixed, revealed by `IntersectionObserver` once the
  controls scroll away (bottom-centre desktop / bottom-left mobile).

## Conventions

- Scope everything under one namespace (`#sl-ap`, `.atlas`, `.slc`).
- Put all editable content (links, cities, copy) in a single `CONFIG` object at
  the top of the file.
- `<noscript>` real-anchor fallbacks where feasible for SEO / no-JS.
- Follow `ios-safari-scroll-rules.md` without exception.
