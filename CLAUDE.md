# Dev Tracker — Claude Context

## What This App Is
A web-based design development tracker for managing styles and factory quotes across multiple brands. Single-file HTML app hosted on GitHub Pages, password protected, no user accounts.

## Live URL
`https://alextoy82.github.io/dev-tracker`

## Local Preview
```
python3 -m http.server 8080
# then open http://localhost:8080
```

## The File
Everything is in one file: `/Users/alex/dev-tracker/index.html`
No build step. Edit the file, push, done.

## How to Push Changes
```
cd /Users/alex/dev-tracker
git add index.html
git commit -m "describe the change"
git push
```
GitHub Pages deploys automatically in ~30 seconds.

## Tech Stack
- **Frontend:** Vanilla JS, single HTML file, no frameworks, no build tools
- **Database:** Supabase (PostgreSQL) — `https://eiykvjfutqdrvwsymdtx.supabase.co`
- **Storage:** Supabase Storage bucket `spec-images`
- **Auth:** Supabase `signInWithPassword` (email + password, single shared account)
- **Hosting:** GitHub Pages — repo `https://github.com/alextoy82/dev-tracker`
- **CDN:** Supabase JS `@2.105.1` from jsDelivr with SRI hash (pinned — do not change version without regenerating the hash)

## Supabase Config (in index.html)
```js
const SUPABASE_URL = 'https://eiykvjfutqdrvwsymdtx.supabase.co';
const SUPABASE_KEY = 'sb_publishable_cD0TuUoyW910GvfW8pmaVg_IDbnRBSC'; // anon/publishable key
const BUCKET = 'spec-images';
```

## Database Tables
- **styles** — one row per style (style_name, style_number, gender, brand, date_passed, status, notes, spec_image_url)
- **quotes** — factory quotes linked to styles (factory_name, factory_contact, date_sent, sample_eta, is_sampling, costings JSONB, image_urls text[], notes)
- **style_comments** — comments per style (factory_name, comment, created_at)

All three tables have RLS enabled. Policy: authenticated users have full access, anon has none.

## Storage Policies (spec-images bucket)
- SELECT: public (anyone can view images)
- INSERT / UPDATE / DELETE: authenticated users only

## Quote Pricing Structure
Quotes support multiple countries via a `costings` JSONB array:
```js
costings: [{ country: 'China', fob_5k: '8.50', ddp_5k: '11.00', fob_10k: '8.00', ... }]
```
Legacy flat fields (`fob_5k`, `ddp_5k`, etc.) also exist for backward compatibility.
The app renders whichever is present. When importing from CSV, build both the flat fields and the `costings` array.

## Brands (hardcoded in filter buttons)
Sharper Image, Primo, Swissmate

## Status Values (hardcoded in select dropdown)
Spec Ready, In Sampling, Sample ETA Confirmed, Sample Received, Revision Requested, Approved, Cancelled

## Gender Values (DB format vs display)
- DB: `Womens` / `Mens`
- Display: `Women's` / `Men's`
- CSV export uses display format — import must convert back

## Key JS Patterns
- `esc(str)` — HTML entity escape, use on ALL user data in innerHTML
- `safeImgUrl(url)` — validates URL is from Supabase domain before rendering in src attribute
- `toast(msg, type)` — show notification (type: 'error' or omit for success)
- `db` — the Supabase client instance
- `allStyles` — in-memory array of all styles, loaded on login
- `allQuotesCache` — lightweight quote cache (style_id, factory_name, sample_eta, is_sampling)
- `currentStyle` — the style currently open in detail view
- `currentQuotes` — full quotes for the current detail view

## Security Decisions Made (do not undo)
- All image URLs rendered via `safeImgUrl()` — validates against Supabase domain, do not bypass
- `esc()` must be called on every DB value placed into innerHTML — no exceptions
- File uploads: magic-byte validation (JPEG/PNG/GIF/WebP only), 10 MB max
- Upload paths use `crypto.randomUUID()` — do not revert to Math.random()
- Login has client-side rate limiting (5 fails = lockout with exponential backoff)
- CDN script has SRI hash — if upgrading supabase-js version, regenerate hash with:
  `curl -sL <new-url> | openssl dgst -sha384 -binary | openssl base64 -A`
- CSP meta tag requires `'unsafe-inline'` for script-src because this is a single-file app

## Features Built
- Style cards (grid + list view) with filter by brand, gender, status
- Search by style name or SKU
- Factory quotes with multi-country pricing (FOB + DDP per MOQ tier)
- Custom MOQ rows per quote
- Sampling toggle + ETA panel on dashboard cards
- Multiple factory images per quote (gallery view)
- Comments per style with factory/buyer/golden sample tags
- Export CSV (all styles or per-style)
- Export PDF (grid or list layout, or per-style detail)
- Import CSV with preview and change detection (matches by SKU then name, non-destructive)
- Styled confirmation modal for all delete actions

## Backup Strategy
No automatic backups (free Supabase plan). Use Export CSV from the dashboard after significant updates.

## Full Documentation
`/Users/alex/Library/Mobile Documents/iCloud~md~obsidian/Documents/My Brain/My Brain/Design/Dev Tracker App/`
- `Dev Tracker App.md` — overview and setup
- `Security Hardening Log.md` — all security changes made 2026-04-28
