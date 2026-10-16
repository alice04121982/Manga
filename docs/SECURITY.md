# Security and privacy blueprint

Three principles drive every decision here:

1. **Least privilege**: every user, service and key can do only what it must. Database rules, not app code, are the last line of defence.
2. **Never trust input**: everything users type, upload or send is treated as hostile until validated on the server.
3. **Defence in depth and no secrets in code**: assume one layer fails, so several others still hold. Keys live in secret stores, never in the repo.

## Threat model for this product

| Risk | Where it shows up | Control |
|---|---|---|
| Account takeover | Login, password reset | Supabase Auth with email verification, strong password rules, optional two-factor, rate limits on login and reset |
| Seeing or editing other people's data | Database | Row Level Security on every table, deny by default, tests that try to break it |
| Script injection (XSS) | Listing titles, descriptions, messages | Escape all output, no raw HTML rendering, strict Content Security Policy |
| Database injection (SQLi) | Search, filters | Parameterised queries only, never string-built SQL |
| Malicious uploads | Listing photos | Allow only JPEG, PNG, WebP, size limits, re-encode server side, strip EXIF (location), scan and moderate |
| Leaked keys | Public GitHub repo | Secret scanning and push protection on, `.env` ignored, service keys never in app bundles |
| Fake or counterfeit listings | Marketplace | Report button, ISBN matching, seller limits for new accounts, review queue |
| Payment fraud | Phase 4 | Stripe Connect, never touch card data, webhook signature checks |
| Scraping and abuse | Public API | Rate limiting, bot protection on sign up and search |
| Personal data exposure | Profiles, addresses | Collect the minimum, never show addresses, encrypt in transit and at rest |

## Hardening checklist

**Repository**
- Turn on GitHub secret scanning and push protection.
- Branch protection on `main`: pull request, passing checks, no force push.
- Dependabot for dependency updates, and a lockfile committed.
- CI runs lint, type check, tests and a secret scan.

**Supabase**
- Row Level Security enabled on every table, with policies written and tested before any data goes in.
- Mobile app uses only the publishable key. The service role key lives only in server functions and is never shipped to a device.
- Storage buckets private by default, public only for approved listing images, with size and type limits.
- Run Supabase's built-in security advisor before every release.

**Web (Vercel)**
- Security headers: Content Security Policy, HSTS, no framing, strict referrer policy.
- Environment secrets set in Vercel, different per environment.
- Validate every input with a schema on the server (for example Zod).

**Mobile**
- Store tokens in the secure keychain, not plain storage.
- HTTPS only, no sensitive data in logs.
- Keep API keys that must exist in the app restricted by app ID.

**Privacy (UK GDPR)**
- Privacy notice, lawful basis recorded, data export and account deletion built in from Phase 2.
- Keep data in a UK or EU region.
- Retention rules: delete drafts, old messages and removed listing photos on a schedule.

**Operations**
- Error monitoring and alerts, audit log for admin actions, tested backups, written incident plan.
- A lawyer reviews terms, seller rules and Online Safety Act duties before payments launch.

## Before launch

Run an independent security review and a test where someone tries to read another user's data through the API. Any failure blocks release.
