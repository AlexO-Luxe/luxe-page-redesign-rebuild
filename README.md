# Luxe — Page Redesign Rebuild

Reusable design system and production builds for the Student Luxe marketing site
(studentluxe.co.uk, Squarespace). Everything here is written to be pasted into
Squarespace **Code Injection** (site-wide) or **Code Blocks** (per page), and
every build follows the iOS/Safari scroll-crash rules in
[`docs/ios-safari-scroll-rules.md`](docs/ios-safari-scroll-rules.md).

## What's in here

| Path | What it is | Where it goes in Squarespace |
|------|------------|------------------------------|
| `header-nav/aperture-squarespace-header.html` | **"Aperture" site header / navigation** — sticky bar, city directory mega-menu (desktop), "lens curtain" menu (mobile). The header rebuild. | Settings → Advanced → Code Injection → **Header** |
| `pages/atlas-cities-page.html` | **"Atlas" browse-all-cities page** — grouped, filterable, searchable city directory built to scale to 40+ cities. | A **Code Block** on the Our Cities page |
| `docs/design-system.md` | Shared palette, type stack, tokens and interaction language. | Reference |
| `docs/ios-safari-scroll-rules.md` | **Mandatory** rules to avoid the Safari/iOS "unexpected error" fast-scroll crash. | Reference — follow for all new site content |

## House style (short version)

- **Palette:** warm ivory paper, ink navy, bronze/gold accent.
- **Type:** `baskerville-display-pt` (serif display) · `dm-sans` (UI) ·
  `adobe-garamond-pro` (body serif) — all loaded site-wide via Adobe Typekit.
- **Namespacing:** every build is scoped (`#sl-ap` / `.atlas` / `.slc`) so it
  never collides with Squarespace theme CSS.
- **Config-first:** links, cities and copy live in a single `CONFIG` / `URLS` /
  `CITIES` object at the top of each file — edit there, nothing else.

## Golden rule

Before shipping **any** HTML/CSS/JS for the marketing site, read
`docs/ios-safari-scroll-rules.md`. The crash it prevents is a hard tab-reload on
real iPhones during fast scrolling and is easy to reintroduce.
