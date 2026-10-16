# Architecture

## Stack at a glance

| Layer | Choice | Why |
|---|---|---|
| Monorepo | pnpm workspaces + Turborepo | One repo, shared code between app and site |
| Native app | Expo (React Native, TypeScript) | One codebase for iOS and Android, camera and barcode scanning built in |
| Website | Next.js (TypeScript) on Vercel | Fast, good for search engines, which matters for edition pages |
| Shared packages | `@manga/ui-tokens`, `@manga/api-types`, `@manga/pricing` | Same colours, types and pricing logic everywhere |
| Backend | Supabase: Postgres, Auth, Storage, Edge Functions | Database, login and file storage in one place with row-level security |
| Search | Postgres full text plus trigram matching, Typesense later | Enough for launch, upgrade when catalogue grows |
| Payments (Phase 4) | Stripe Connect | Stripe holds the money, not us |
| Images | Supabase Storage with resizing, EXIF stripped on upload | Faster pages, no leaked photo locations |
| Observability | Sentry for errors, Vercel analytics | Catch bugs before users report them |
| CI | GitHub Actions | Lint, type check, tests, secret scan on every pull request |

This matches tools you already use (Vercel, Supabase, GitHub), so there is less new to learn.

## Folder layout

```
manga/
  apps/
    mobile/        Expo app
    web/           Next.js site
  packages/
    ui-tokens/     colours, type, spacing, manga style tokens
    api-types/     types generated from the database
    pricing/       suggested price and offer ranking logic, unit tested
  supabase/
    migrations/    every database change, versioned
    functions/     server-side code (price suggestion, moderation hooks)
  docs/
```

## Data model (simplified)

- **series**: title, original title, author, genres
- **edition**: belongs to a series. `format` is one of single, two_in_one, three_in_one, omnibus, box_set. Has ISBN, publisher, language, volume range, cover.
- **profile**: public seller name and avatar. Private details kept separate.
- **listing**: belongs to an edition and a seller. Price, condition, photos, status (draft, active, sold, removed), postage.
- **offer_source**: a shop and how we get its prices (affiliate feed, API).
- **shop_offer**: edition, source, price, postage, link, last checked.
- **price_snapshot**: daily price per edition, feeds history and the suggestion model.
- **saved_item** and **price_alert**: per user.
- **report**: user flags a listing or profile for review.

The comparison list on an edition page is the union of active `listing` rows and fresh `shop_offer` rows, sorted by total cost (price plus postage).

## Aggregation pipeline

Runs as scheduled server jobs (Supabase Edge Functions and scheduled tasks, moving to a queue worker if volume grows). One adapter per source, all producing the same normalised record.

```
source adapter (feed / API / partner file)
  -> raw_import     untouched copy of what we received
  -> match          ISBN exact, then fuzzy title + volume + format
  -> shop_offer     upsert, with price, postage, stock, last_checked
  -> review_queue   low-confidence matches wait for a human
```

- Tables added: `raw_import`, `match_review`, and `source_run` (when each source last ran, how many items, errors).
- Each adapter is isolated so one broken source cannot stop the others.
- Offers not refreshed within a set window are hidden, not deleted.
- Outbound buy links carry affiliate tags where the source allows it, and are logged as clicks for reporting.
- Source credentials live in secret stores, one per source, rotated on a schedule.
- Feed content is untrusted: titles and descriptions are escaped and length limited before display.

## Suggested price (v1, rules based)

1. Anchor: lowest and median fresh shop price for the same edition.
2. Used discount by condition (new, like new, good, acceptable), tunable in one config file.
3. Adjust using comparable active listings for the same edition.
4. Return a low, suggested and high price with a short reason ("12% under the cheapest new copy").
5. Later, once there are enough sold prices, replace steps 2 and 3 with a learned model. The logic lives in `packages/pricing` so the app, site and server give the same answer.

## Barcode listing flow

Scan the ISBN barcode, match to an `edition`, pre-fill title and format, user adds photos, condition and price. If no match, the user submits it for catalogue review rather than typing free text, which keeps the catalogue clean.

## Environments

- Local, Preview (every pull request gets a Vercel preview and a Supabase branch), Production.
- Database changes only through migrations, never edited by hand in production.
