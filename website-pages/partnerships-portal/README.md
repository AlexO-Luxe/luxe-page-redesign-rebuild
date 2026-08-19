# Guest Services portal

Guest-only area on studentluxe.co.uk. Sign in with booking reference +
surname, browse partner perks. Built as "The Deck" (option B).

## Files

| File | What it is |
|------|-----------|
| `guest-services-live.html` | **The one to paste.** Squarespace Code Block, talks to the API. |
| `option-a-the-book.html` | Design option A, demo data, not wired. Keep for reference. |
| `option-b-the-deck.html` | Design option B, demo data. The mockup this was built from. |
| `option-c-the-club.html` | Design option C, demo data, not wired. |

Backend lives in **`AlexO-Luxe/luxe-lead-capture`**:

| File | What it is |
|------|-----------|
| `api/guest-session.js` | `POST /api/guest-session` — verifies ref + surname against the Bookings board, returns a 12 hour session token |
| `api/guest-partners.js` | `GET /api/guest-partners` — returns partners from the Monday partners board, cached 5 minutes |
| `api/_guest-auth.js` | Token signing, CORS allowlist, rate limiting, Monday client |
| `scripts/guest-auth-check.js` | `node scripts/guest-auth-check.js` — offline self-check on the auth path |
| `scripts/guest-session-live-check.js` | `node scripts/guest-session-live-check.js <ref#> <surname>` — signs in against real Monday, proves every column still resolves |

---

## Setup, in order

### 1. Create the Monday partners board

Columns are matched by **title**, not id, so build it in the Monday UI and
nobody ever has to read a column id. Item name is the partner name.

| Column | Type | Example |
|--------|------|---------|
| (item name) | — | `Third Space, Soho` |
| Category | Status or Text | `Fitness` |
| City | Status or Text | `London`, or `All cities` for anything nationwide |
| Perk | Text | `20% off membership` (short, it renders as a badge) |
| Code | Text | `LUXE-TS20` (leave blank and the card says "No code needed") |
| Description | Long text | One or two lines, shown on the card |
| Terms | Long text | Small print, revealed with the code |
| Link | Link or Text | `https://...` (anything not http/https is dropped) |
| Image URL | Text | Squarespace CDN url, `?format=750w` is added automatically |
| Status | Status | **`Active` publishes the row.** Anything else stays hidden |
| Order | Numbers | Optional, low numbers first |

Categories and city filters in the portal are generated from whatever is on
the board, so adding a category is just typing it in a row.

### 2. How the reference resolves (no config needed)

The number guests hold is the **Ref #**: Leads board column `item_id7`
("Ref # (AO in use)"), which is that lead's Monday item id. Booking Flow shows
the same number as the mirror column `mirror8`.

Sign in therefore goes **lead first**:

```
digits(typed ref) -> Leads item -> connect_boards75 -> Booking Flow rows -> any live?
```

Two Monday facts force that shape, both confirmed against the live API:

- A **mirror column cannot be filtered** with `items_page` `query_params`, so
  searching Booking Flow by `mirror8` is impossible.
- A mirror returns its value in **`display_value`**, not `text`. Read only
  `text` and it comes back `null`.

Fetching the lead by id needs no search at all, so sign in is one API call. If
the number is not a lead, it is retried as a Booking Flow item id, which covers
the booking-side "Alex Booking Ref" column (`pulse_id_mm3t29qp`) should that
ever become the number printed on confirmations.

Guests can type `9038612423`, `SL-9038612423` or `Ref 9038612423`: only the
digits are used. The confirmation email has to show that Ref #, or guests have
nothing to type. That is the one dependency outside this build.

**One lead, several bookings** is normal (an extension, a rebooking, a
cancelled row beside a live one). Access is granted when *any* linked booking
is live, and the profile shows the live booking with the latest check in
(`date69`).

### 3. Vercel env vars (luxe-lead-capture project)

| Var | Required | Notes |
|-----|----------|-------|
| `GUEST_PORTAL_SECRET` | yes | HMAC key for session tokens. Generate with `openssl rand -hex 32`. Rotating it signs everyone out. |
| `PARTNERS_BOARD_ID` | yes | The board from step 1 |
| `GUEST_PORTAL_ORIGINS` | recommended | Comma list. Defaults to `https://www.studentluxe.co.uk,https://studentluxe.co.uk` |
| `GUEST_BLOCKED_STATUSES` | optional | Booking Stage labels that cannot sign in. Default `Lost Booking,Cancelled Booking,Pending Booking`. Matched **exactly**, so `Extensions - Pending` stays allowed. Drop `Pending Booking` from the list if unconfirmed guests should get in |
| `GUEST_ACCESS_GRACE_DAYS` | optional | Days after check out that access survives. Default `30`. Booking Stage has no "checked out" label, so this is the only thing stopping a guest from 2024 still pulling live codes |
| `GUEST_LEADS_BOARDS` | optional | Default `2171015719,3265428349` (current + legacy leads) |
| `GUEST_BOOKING_BOARDS` | optional | Default `2171015589` (Booking Flow) |
| `GUEST_LEAD_BOOKING_RELATIONS` | optional | Lead → booking relation columns. Default `connect_boards75` |
| `GUEST_SESSION_TTL_SECONDS` | optional | Default `43200` (12 hours) |
| `PARTNERS_CACHE_SECONDS` | optional | Default `300` |

`MONDAY_API_KEY`, `CRON_SECRET` and the Upstash Redis vars are already set on
that project.

### 4. Squarespace

1. New page, e.g. `/guest-services`. Add a Code Block, paste
   `guest-services-live.html`.
2. Set the section background to cream `#FBF8F1`.
3. In the block, set `CONFIG.API_BASE` to the Vercel domain.
4. Page settings → SEO → **hide from search engines**. Gated content
   should never index.

### 5. Test

```bash
curl -s -X POST https://<vercel-domain>/api/guest-session -H 'Content-Type: application/json' -d '{"ref":"<a real Ref # from Monday>","surname":"<that guest surname>"}'
```

Expect `{"token":"...","guest":{...}}`. Then:

```bash
curl -s https://<vercel-domain>/api/guest-partners -H 'Authorization: Bearer <token from above>'
```

Wrong details must return `401` with a generic message, never a hint about
which half was wrong.

---

## How it behaves

- **Session**: token in `sessionStorage`, sent as `Authorization: Bearer`.
  No cookies, so Safari third-party cookie blocking is a non-issue. Closing
  the tab signs the guest out; reloading the page does not.
- **Rate limits**: 10 sign in attempts per IP per 15 minutes, 20 per booking
  reference per hour. Fails open if Redis is down.
- **Surname matching**: case, spaces, punctuation and accents ignored. Matches
  the lead's last name, first name, or any word of the row title, so
  "Smith, Jane, Chelsea" accepts "Smith". Checked *before* booking state is
  revealed, so a wrong surname and a cancelled booking look identical from
  outside.
- **Access window**: a booking counts as current when its check out
  (`date_1`), or its check in when there is no check out, is no more than
  `GUEST_ACCESS_GRACE_DAYS` in the past. Past guests get the 403, not the
  portal. Widen the window with that env var, do not remove it.
- **Partner cache**: 5 minutes. Force a refresh with
  `GET /api/guest-partners?refresh=<CRON_SECRET>` — the secret alone is
  enough, no guest token needed.
- **Tabs**: "About my stay", "Payments" and "Support" are static coming soon
  panels driven by `CONFIG.sections` in the block. Turn one live by building
  its section and flipping `state:'soon'` to `state:'live'`.

## Not built yet

- The three coming soon sections (they are panels, not features).
- Any write path: nothing in the portal edits Monday.
- Personalisation by city. The API returns every active partner and the guest
  filters; the token does carry the guest's city if we want to default that
  filter later.

## Security notes

Booking reference + surname is weak authentication and is treated that way:
short sessions, hard rate limits, generic errors, and nothing behind the gate
beyond partner discount codes. Do not put payment details, documents or
addresses behind this login without a stronger second factor (a one time code
to the email on the booking is the natural next step, and is what "Payments"
needs before it goes live).

Tracking is untouched: this block contains no GTM, Consent Mode or UTM code.
