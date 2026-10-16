# Product plan

## The idea

One place to find a manga, see exactly what edition it is, and know where to buy it and for how much. People can also sell their own copies, with a suggested price so they are not guessing.

## Who it is for

- Buyers who want to know "which edition is this, and where is it cheapest?"
- Collectors who want the right edition (omnibus vs single volumes) and fair prices.
- Sellers clearing a shelf who do not know what a set is worth.

## Core features

1. **Catalogue**: every title broken into Series, Edition and Volume. An edition has a format: single, 2-in-1, 3-in-1 omnibus, box set. Cover image, publisher, ISBN, language, genre.
2. **Price comparison**: for each edition, a list of offers (user listings and shop prices) sorted cheapest first, with a "best offer" tag, postage shown, and condition for used copies.
3. **Listings**: sellers take photos, pick the edition (scan the barcode to find it), choose condition, set a price.
4. **Suggested price** (like Airbnb): the app proposes a price range from comparable listings and shop prices. The seller can accept or override it.
5. **Search and filters**: by title, genre, format, condition, price, language.
6. **Accounts, saved items, price alerts**.
7. **Aggregation from other sites**: pull manga listings from the shops and marketplaces that already sell them, match each one to the right edition, and show them in one place with a link out to buy. See "Aggregation" below.

## Design direction

Manga visual language: bold black ink lines, screentone dot patterns, speech-bubble callouts, panel-style grids, high contrast. The risk is it becoming noisy, so the rule is: strong style on headings, covers and empty states, calm and legible on prices and forms.

Reference patterns found in Mobbin:

- Price comparison list with a "Best offer" tag and shipping under each seller: [Klarna](https://mobbin.com/screens/59d801e4-226c-4c2c-b5a0-cae4b19ce509) and [second Klarna screen](https://mobbin.com/screens/b46cb931-a989-418d-8d76-32f137e2c43e)
- Used item page with condition, price and description: [Vestiaire Collective](https://mobbin.com/screens/4732384a-c8ef-4007-bfa6-395eb0cdd672)
- Sell flow (photos, title, category, condition, price, upload): [Vinted Sell](https://mobbin.com/flows/f6d5c7c8-cc36-4fdb-9896-e6e93476f22e) and [Depop listing](https://mobbin.com/flows/9f6278f3-41a4-4e03-9e3f-0db7cc5bcac7)
- "Visually similar" and "similar items" rails: [Etsy](https://mobbin.com/screens/58eff559-84eb-4474-bb92-a40d11377654) and [Alta](https://mobbin.com/screens/29fec1d4-7ff3-4a8e-8965-ce5e610ced81)
- Seller shop with earnings and active listings: [Depop shop](https://mobbin.com/flows/9f6278f3-41a4-4e03-9e3f-0db7cc5bcac7)

Impeccable skill: not found among the skills connected to this workspace. Closest available are `design-taste-frontend` and `frontend-design`. Needs a decision, see open questions.

## Existing landscape

A quick search found general price comparison sites (for example PriceSpy lists some manga) and several small collection trackers, but no manga-specific marketplace with price comparison. This was a light check, not market research. Worth a proper competitor pass before launch.

## Aggregation: one place for every manga on sale

Goal: someone searches a title once and sees every copy for sale across many sites, not only our own sellers.

How we get the data, in order of preference:

1. **Affiliate and product feeds**: retailers and marketplaces that publish catalogues to affiliate networks. Legal, structured, and we earn a commission on clicks. Which retailers offer manga feeds needs checking.
2. **Official APIs**: some marketplaces offer search APIs. Their terms often limit how prices can be shown, so each one is checked before use.
3. **Direct partnerships and shop-submitted feeds**: independent manga shops can send a spreadsheet or connect their shop platform. Good for niche and import stock.
4. **No scraping without permission**: copying pages from shop sites usually breaks their terms, breaks whenever they change layout, and risks legal trouble and being blocked. If we ever do it, only for sites that agree in writing.

The pipeline, in plain terms:

1. **Collect**: fetch new and changed items from each source on a schedule.
2. **Match**: work out which edition each item is. ISBN match first, then a fuzzy match on title, volume and format. Anything uncertain goes to a review queue instead of guessing, because a wrong match shows the wrong price.
3. **Clean**: remove duplicates, normalise currency, postage and condition.
4. **Show**: add it to the edition's comparison list with the shop name, total price, and a link to buy there.
5. **Keep fresh**: re-check prices regularly and hide items that are stale or out of stock.

What this changes:

- We do not hold stock or take payment for aggregated items, the buyer completes the purchase on the other site. That keeps this part simple.
- It solves the cold start problem: the comparison list is useful on day one without any user listings.
- Mark aggregated items clearly as "sold by [shop]" and disclose affiliate links.
- You said "magazines": I have treated this as manga books (single volumes, omnibuses, box sets). If you also mean serialised magazines such as weekly or monthly issues, that is a separate item type to add to the catalogue.

## Phases

| Phase | Goal | Done when |
|---|---|---|
| 0. Foundations | Repo, monorepo, CI, Supabase project, design tokens | Empty app and site deploy from main |
| 1. Catalogue | Browse and search editions, edition detail pages | You can find any seeded title and see its formats |
| 2. Aggregation | Ingest and match listings from the first 2 to 3 retailer feeds | Edition pages show real offers from other sites with buy links |
| 3. Listings | Sign up, scan barcode, photo upload, post a listing | A real user listing appears on its edition page |
| 4. Suggested price and alerts | Ranked offers, price suggestion, price alerts | Edition page shows a suggested range and sellers can accept it |
| 5. Trust and payments | Messaging, reporting, Stripe Connect checkout, reviews | A buyer can pay a seller without us holding funds |
| 6. Polish and launch | Accessibility, performance, store submission | App Store and Play Store approved |

Recommendation: ship Phases 1 to 4 first with no payments (buyers follow a link or contact sellers). Aggregation comes before user listings because it gives the app value from day one. Payments bring the heaviest legal, fraud and support load, so earn demand first.

## Hard problems to plan for

- **Cold start**: no listings on day one means no useful price comparison. Seed with shop prices and a small set of popular series, and invite a first group of sellers.
- **Suggested price needs data**: until there are sold prices, the suggestion is rules based (shop price, condition discount, comparable listings). A learned model comes later.
- **Retailer prices**: use affiliate feeds or official APIs. Do not scrape shop sites, it breaks their terms and breaks often. Which retailers offer feeds needs checking.
- **Cover images and metadata**: covers are copyrighted. Use sellers' own photos, and metadata and image sources whose licence allows it. Needs checking before launch.
- **Counterfeit and bootleg manga**: a known problem on resale sites. Needs reporting tools, ISBN matching and seller rules from the start.
- **Mature content**: some manga is adult. Age gating and content rules are needed.
- **UK rules**: UK GDPR, consumer law for sellers, and Online Safety Act duties for platforms with user content. Get a lawyer's review before Phase 4.

## Open questions for you

1. Launch country and currency: UK and GBP first?
2. Is Phase 1 to 4 without payments acceptable, with aggregation first?
3. Impeccable: what is it exactly, and how should I load it? Otherwise I use `design-taste-frontend`.
4. App name and domain (I can check availability).
5. New Supabase and Vercel projects under your accounts, or reuse existing ones?
