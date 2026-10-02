# DentraFin
# Dentist earnings app: project foundation

**Status:** the production build passes and the 6 engine tests pass. Ported from the prototype: login, Add (NHS/Private switch, treatment carousel, lab fee, initials), Log (tap to edit popup with added/last-modified, ✕ delete), Earnings (pay periods, weeks and best days, tap-to-see-activity popups), Referrals, Settings (practices, bands, light/dark/auto and primary/secondary colours). **Not yet ported: the Accounts tab** (uploads, quarterly figures, steps, disclaimer); the `/api/extract` route exists but has no screen. Nothing has been run against a live Supabase project yet, so expect small fixes on first run. Supabase returns at most 1,000 entries per request by default, so add paging before real use. Treatments, NHS cut-off dates, pay day and tax % have no editing screen yet; change them in the Supabase table editor for now.

## What's in this folder
| File | What it is | Tested? |
|---|---|---|
| `supabase/schema.sql` | Tables for settings, practices, treatments, entries, referrals, actual pay and accounting records. Row Level Security so each user sees only their own data. A sign-up trigger creates default settings, one practice and the starter treatment list. | Not run yet |
| `src/lib/engine.ts` | Earnings maths from the prototype: marginal monthly private bands per practice, UDA income, lab share, NHS cut-off periods, pay dates. | Yes: 6 tests pass |
| `src/lib/engine.test.ts` | Tests for the engine. Add a test for every real payslip you check against. | Yes |
| `src/app/api/extract/route.ts` | Server route that reads a PDF or photo with Claude (your API key, never exposed to the browser). | **No.** Try with real documents. |

## Setup, in order
1. **Supabase:** create a free project at supabase.com. In SQL Editor, run `supabase/schema.sql`.
2. **Next.js:** `npx create-next-app@latest dentist-app` (TypeScript, App Router, Tailwind). Copy `src/` and `supabase/` into it, then `npm i @supabase/supabase-js @supabase/ssr @anthropic-ai/sdk` and `npm i -D vitest@3`.
3. **Environment variables** (`.env.local`, and later in Vercel): `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `ANTHROPIC_API_KEY`. Never commit these.
4. **Login:** Supabase > Authentication > Providers. Enable Email first, then Google (Google Cloud OAuth client). Apple needs a paid Apple Developer account, a Services ID and a signing key, and the Apple secret has to be regenerated about every 6 months.
5. **Deploy:** push to GitHub, import into Vercel, add the same environment variables.
6. **Check the maths** against real payslips with your wife before anyone else uses it.

## Still to build (suggested order)
1. Auth pages and a protected app shell, with a responsive layout: bottom tabs on phones, a sidebar on desktop.
2. Add / Log screens (NHS / Private switch, initials, practice picker) writing to `entries`.
3. Earnings screens using `calc()` and `periods()`, with actual-pay comparison.
4. Settings (practices, bands, treatments, cut-offs) and personalisation.
5. Referrals.
6. Accounts tab: upload to `/api/extract`, review, quarterly figures, CSV export, steps and disclaimer.
7. Account deletion and data export.

## Before charging anyone
- Register with the ICO and publish a privacy policy and terms (UK GDPR). Decide whether uploaded documents are discarded after reading (as now) or stored.
- Add rate limits to `/api/extract`. Each upload costs you an API call.
- Stripe for subscriptions (UK VAT and invoices).
- Have an accountant review the tax wording and the disclaimer.
- The app prepares Making Tax Digital figures only. Sending updates to HMRC directly needs HMRC recognition and is a separate project.
- Re-check tax thresholds, deadlines and rates each April.

## Putting it on GitHub
```
cd your-project-folder
git init && git add . && git commit -m "First version"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git push -u origin main
```
Check `.env.local` is listed in `.gitignore` (create-next-app does this) so your keys are never uploaded. If you re-run `schema.sql` after changes, drop the old tables first or run the changes as a migration.
