# External Integrations

**Analysis Date:** 2026-03-22

## APIs & External Services

**Database & Authentication:**
- Supabase - PostgreSQL backend + user authentication
  - SDK: `@supabase/supabase-js` 2.94.1, `@supabase/ssr` 0.8.0
  - Auth: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` (client), `SUPABASE_SERVICE_ROLE_KEY` (server)
  - Used in: `lib/supabase/client.ts`, `lib/supabase/server.ts`
  - Client: `createBrowserClient()` for CSR/SPA, `createServerClient()` for SSR with cookie handling

**Payment Processing:**
- Stripe - Subscription and billing management
  - SDK: `stripe` 20.4.0
  - Auth: `STRIPE_SECRET_KEY` (server-only), `STRIPE_PRICE_ID`
  - Used in: `app/api/billing/create-checkout-session/route.ts`, `app/api/billing/portal/route.ts`
  - Implements: Customer creation, checkout session flow, billing portal
  - Metadata: Stores `stripe_customer_id` in Supabase auth user metadata

- Paystack - Alternative payment processor (not fully integrated in app routes)
  - Auth: `PAYSTACK_API_KEY` (referenced in migrations), plan codes stored in Supabase
  - Used in: `app/api/billing/paystack/initialize/route.ts`, admin subscription management
  - Metadata: Stores `paystack_customer_id`, `paystack_subscription_code` in Supabase

**Email Service:**
- Resend - Transactional email delivery
  - SDK: `resend` 4.8.0
  - Auth: `RESEND_API_KEY` (server-only)
  - Used in: `app/api/contact/route.ts`
  - Implements: Contact form email delivery to `CONTACT_EMAIL`

**Data Import & Sync:**
- Airtable - Content management (questions, topics, taxonomy)
  - Auth: `AIRTABLE_API_KEY`, `AIRTABLE_BASE_ID`
  - Tables: `Questions_v2`, topic taxonomy, microtopics, engine metadata
  - Used in: Multiple scripts in `scripts/` directory
  - Sync direction: Airtable → Supabase (via npm scripts)
  - Scripts:
    - `scripts/sync-airtable-to-supabase.mjs` - Sync questions and taxonomy
    - `scripts/publish-airtable-to-supabase.mjs` - Publish approved questions
    - `scripts/backfill-airtable-engine-metadata.mjs` - Sync engine metadata
    - `scripts/backfill-lead-in-from-airtable.mjs` - Sync question lead-in text
    - `scripts/sync-airtable-engine-tags.mjs` - Sync learning engine tags
    - `scripts/export-airtable-questions-v2.mjs` - Export questions v2
    - `scripts/audit-airtable-questions-v2.mjs` - Validate question data
    - `scripts/migrate-airtable-v1-to-v2.mjs` - Migration script

**Performance Monitoring:**
- Vercel Speed Insights - Client-side performance monitoring
  - Package: `@vercel/speed-insights` 1.3.1
  - Used in: `app/layout.tsx` (imported as `<SpeedInsights />` component)
  - Collects: Core Web Vitals metrics

**Fonts:**
- Google Fonts (via Next.js)
  - Used in: `app/layout.tsx`
  - Fonts: Manrope, Space Grotesk (loaded at build time)

## Data Storage

**Databases:**
- PostgreSQL (Supabase hosted)
  - Connection: `NEXT_PUBLIC_SUPABASE_URL` (public), configured in `next.config.ts`
  - Client: Supabase JavaScript SDK (not traditional ORM - uses RPC and direct queries)
  - Migrations: `supabase/migrations/` (21 migration files)
  - Schema: Questions, attempts, assessments, users, subscriptions, topics, taxonomy

**File Storage:**
- Local filesystem only - No cloud storage integration
- Excel/PDF handling: Client-side via `xlsx` and `pdf-parse` packages

**Caching:**
- None explicitly configured - Relies on Next.js default caching and Supabase client-side caching

## Authentication & Identity

**Auth Provider:**
- Supabase Auth - Custom auth with email + password
  - Implementation: Server-side session with cookies (via `createServerClient()`)
  - Auth routes: `app/api/auth/logout/route.ts`, login/register pages
  - User metadata: Stores `stripe_customer_id`, `paystack_customer_id`
  - Email verification: Built-in to Supabase

## Monitoring & Observability

**Error Tracking:**
- Console logging only (no Sentry, Rollbar, or equivalent)
- Error boundaries: `app/error.tsx` (Next.js error boundary)

**Logs:**
- Console logs in API routes (e.g., Resend errors in contact form)
- Request/response logging: Not centralized

## CI/CD & Deployment

**Hosting:**
- Vercel (primary web app) - Referenced in `app/api/billing/create-checkout-session/route.ts` with `NEXT_PUBLIC_VERCEL_URL`
- Electron Desktop - Standalone builds for Windows/macOS/Linux
  - Build commands: `npm run electron:build:win`, `electron:build:mac`, `electron:build:linux`
  - Configured in: `electron-builder.json`

**CI Pipeline:**
- None explicitly configured in repo
- GitHub Actions workflows: `workflows/` directory exists but not detailed
- Vercel auto-deployment from git push

**Build Process:**
- `npm run build` - Next.js production build
- `npm run lint` - ESLint validation
- `npm run typecheck` - TypeScript type checking
- `npm run ci:verify` - Full validation (lint + typecheck + build)

## Environment Configuration

**Required env vars:**
- `NEXT_PUBLIC_SUPABASE_URL` - Supabase project URL (public)
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Supabase anonymous key (public)
- `SUPABASE_SERVICE_ROLE_KEY` - Supabase service role key (server-only, sensitive)

**Optional env vars:**
- `RESEND_API_KEY` - Email service activation
- `CONTACT_EMAIL` - Email recipient for contact form (defaults to `bernhardt@medi-quiz.com`)
- `STRIPE_SECRET_KEY` - Stripe billing activation
- `STRIPE_PRICE_ID` - Stripe subscription price
- `NEXT_PUBLIC_VERCEL_URL` - Deployment URL (auto-set by Vercel)
- `NEXT_PUBLIC_APP_BASE_URL` - Custom app base URL override
- `AIRTABLE_API_KEY` - Airtable API access (scripts only)
- `AIRTABLE_BASE_ID` - Airtable base ID (scripts only)
- `AIRTABLE_TABLE_NAME` - Airtable table name (defaults to `Questions_v2`)
- `AIRTABLE_VIEW_NAME` - Airtable view filter (optional)

**Secrets location:**
- `.env.local` - Local development (git-ignored)
- Vercel environment settings - Production/staging

## Webhooks & Callbacks

**Incoming:**
- Stripe webhooks - Not implemented in current routes (would need `app/api/webhooks/stripe/route.ts`)
- Paystack webhooks - Not implemented in current routes (would need webhook endpoint for subscription confirmation)

**Outgoing:**
- Stripe redirect URLs: Success/cancel callbacks from checkout session
  - Success: `/profile/subscription?stripe=success`
  - Cancel: `/profile/subscription?stripe=cancel`
- Paystack redirect URLs: Success/failure callbacks
  - Success: `/profile/subscription?paystack=success`

---

*Integration audit: 2026-03-22*
