# Good Beer Here

An anonymous, crowdsourced map of pubs, taprooms and breweries that genuinely have good beer.

Not a star-rating app. A venue either has enough independent, verified reports to earn its place on the map — or it doesn't appear yet. The app never publishes a negative claim about any named business; a venue simply isn't shown until it clears the bar. That's a deliberate design principle, not a side effect, and any future feature should be checked against whether it breaks this positive-only guarantee.

## What counts as "good beer"

Locally produced, well-kept cask ale — not the big national brands (London Pride, Doom Bar) — plus genuine independent craft beer, not mass-market lager (Stella, Carlsberg). Some "craft-labelled" beers are now owned by large drinks companies too (Beavertown, for example, is owned by Heineken); the app treats those the same as any other big-brand beer.

## How it works

1. **Find a venue** (`find-venue.html`) — pulls real, live venues from OpenStreetMap based on your current location. Nothing hardcoded.
2. **Report on it** (`report-form.html`) — a single question: what's actually on offer, as a select-all-that-apply (Craft Cask / Craft Keg / Craft Bottles-Cans). GPS is captured and checked against the venue's real coordinates — a report is rejected if you're more than 150m away, so reports can't be submitted from home or "on the way."
3. **If the venue isn't in the database yet**, but OpenStreetMap knows about it, submitting a report **creates it** — using OSM's coordinates as the source of truth, not the reporter's own position. You just pick which of six venue types it is.
4. **If OpenStreetMap doesn't know about it either**, it can be flagged as a possible new venue instead (saved to a holding table, `pending_venues`) for manual confirmation later.
5. **The homepage** (`index.html`) reads venues live from the database and shows them on a map with cards, including a direct "Report here" link for each one.

## Consensus rule

A venue is considered to have real, independent confirmation once it has **4 independent reports, at least 30 days apart** (not just a flat count — this avoids biasing toward busy city venues over quiet, well-loved village pubs, since it responds to time rather than trying to guess footfall).

**Current MVP behaviour:** all venues are shown transparently with their report count and recency, whatever it is — nothing is hidden. The plan is to later "flip a switch" and only show venues that clear the 4/30 threshold, once there's enough real report volume for that to make sense. The underlying rule doesn't change — only what's gated by it.

## Founder Picks

A handful of venues were seeded directly by the founder to give the app value before organic data existed. These are clearly labeled "Founder Pick," distinct from organically-verified venues — never quietly dressed up as crowd consensus.

## Tech stack

- **Hosting:** GitHub Pages (free, static)
- **Database:** Supabase (hosted Postgres, free tier)
- **Venue data:** OpenStreetMap via the Overpass API (free, no key)
- **Map rendering:** Leaflet.js
- No user accounts anywhere — reporting is fully anonymous. Only the founder's own Supabase login has any elevated access.

## Files

| File | Purpose |
|---|---|
| `index.html` | Public homepage — map, venue cards, live data from Supabase |
| `find-venue.html` | OSM-based nearby venue lookup + "flag as new venue" form |
| `report-form.html` | The report/checklist screen, GPS verification, venue auto-creation |
| `manifest.json` | Makes the site installable as a home-screen app |
| `icon-192.png` / `icon-512.png` | App icon |

## Database structure (Supabase)

- **`venues`** — id, name, area, type, lat, lng, note, is_founder_pick, created_at
- **`reports`** — id, venue_id, craft_cask, craft_keg, craft_bottles_cans, report_lat, report_lng, created_at
- **`pending_venues`** — id, name, type, lat, lng, created_at (holding table for venues not found on OSM)

Row Level Security is enabled on all three tables. Public visitors can read venues/reports and insert new ones; only an authenticated (founder) login can update or delete.

## Known limitations / open questions

- No admin login yet — managing venues, and confirming `pending_venues`, currently means going into Supabase's Table Editor by hand.
- Auto-created venues (from the OSM flow) have a blank `area` field.
- No rate-limiting / anti-abuse beyond the GPS check — e.g. nothing currently stops the same device reporting on the same venue repeatedly.
- No resolution mechanism yet for genuinely disputed/conflicting reports.
- "Well kept" (cask condition) was dropped from the MVP checklist as too hard for a non-expert to judge reliably — may return in some form later.
- No privacy policy / terms of service yet, despite collecting live GPS data (anonymously).
- Free-tier Supabase has **no automatic backups**. Current safety net: the SQL scripts used to build the schema and seed data are saved and re-runnable; real crowd report data is not currently backed up anywhere else.

## Roadmap (rough order)

1. Tie the three pages together with consistent navigation
2. Build the report-count/recency display on the homepage
3. Build a proper interface for reviewing the `pending_venues` queue
4. Real admin login, replacing manual Table Editor work
5. Rate-limiting / anti-abuse
6. Privacy policy / ToS
7. Revisit the consensus "gating switch" once there's real report volume