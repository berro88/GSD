# Codebase Structure

**Analysis Date:** 2026-03-22

## Directory Layout

```
D:/dev/medi-quiz-frontend/
├── app/                          # Next.js App Router (pages, routes, layouts)
│   ├── layout.tsx               # Root layout (header, footer, navigation)
│   ├── page.tsx                 # Home page
│   ├── error.tsx                # Error boundary
│   ├── globals.css              # Global styles
│   ├── favicon.ico              # Favicon
│   │
│   ├── api/                     # API routes
│   │   ├── admin/               # Admin operations
│   │   │   ├── users/           # User management routes
│   │   │   ├── attempts/        # Attempt grading routes
│   │   │   ├── questions/       # Question publishing routes
│   │   │   └── subscription/    # Subscription management
│   │   ├── auth/                # Authentication endpoints
│   │   │   ├── status/route.ts  # Check auth status
│   │   │   └── logout/route.ts  # Logout handler
│   │   ├── attempts/            # Quiz attempt operations
│   │   │   ├── save/route.ts    # Save answers
│   │   │   └── start/route.ts   # Create new attempt
│   │   ├── billing/             # Payment and subscription
│   │   │   ├── paystack/        # Paystack integration
│   │   │   └── stripe/          # Stripe integration
│   │   ├── config/              # Public configuration
│   │   ├── contact/             # Contact form submission
│   │   ├── register/            # User registration
│   │   └── webhook/             # External webhook handlers
│   │
│   ├── admin/                   # Admin UI pages
│   │   ├── questions/page.tsx   # Question publishing interface
│   │   └── users/page.tsx       # User management interface
│   │
│   ├── analytics/               # Analytics dashboard page
│   ├── assessments/             # Assessment browsing and startup
│   │   ├── page.tsx             # Assessment catalog
│   │   ├── AttemptInitForm.tsx  # Form to configure attempt
│   │   └── StartAttemptButton.tsx
│   │
│   ├── attempts/                # Quiz attempt pages
│   │   ├── page.tsx             # Attempts history/list
│   │   └── [attempt_id]/        # Single attempt pages
│   │       ├── page.tsx         # Main quiz UI
│   │       ├── timeout/page.tsx # Attempt timed out
│   │       ├── review/page.tsx  # Review submitted attempt
│   │       ├── AnswerPicker.tsx # Answer selection component
│   │       ├── AbandonAttemptButton.tsx
│   │       ├── ResetAttemptButton.tsx
│   │       ├── ElapsedDuration.tsx # Timer display
│   │       ├── TimeRemaining.tsx    # Time left warning
│   │       ├── AttemptQuestionsOneByOne.tsx # Question carousel
│   │       └── review/ReportIssueForm.tsx
│   │
│   ├── dashboard/               # Main dashboard page
│   ├── profile/                 # User profile pages
│   │   ├── page.tsx             # Profile editor
│   │   └── subscription/        # Subscription management
│   │
│   ├── login/page.tsx           # Login page
│   ├── register/page.tsx        # Registration page
│   ├── contact/page.tsx         # Contact form page
│   ├── terms/page.tsx           # Terms of service
│   ├── privacy/page.tsx         # Privacy policy
│   │
│   └── PresencePinger.tsx       # Real-time presence tracker component
│
├── lib/                         # Shared utilities and business logic
│   ├── api/                     # Client-safe API fetches
│   │   ├── assessments.ts       # Assessment listing
│   │   ├── attempts.ts          # Attempt queries
│   │   ├── attempts.client.ts   # Client-only attempt fetches
│   │   ├── events.client.ts     # Event tracking
│   │   ├── presence.client.ts   # Presence tracking
│   │   └── issues.client.ts     # Issue reporting
│   │
│   ├── server/                  # Server-only functions (use "server-only" directive)
│   │   ├── analytics/
│   │   │   └── getMyAnalytics.ts        # User performance analytics
│   │   ├── attempts/
│   │   │   ├── getAttempt.ts           # Full attempt + answers
│   │   │   ├── getAttemptMeta.ts       # Attempt metadata and timing
│   │   │   ├── getAttemptQuestionIds.ts
│   │   │   └── getReview.ts            # Attempt review data
│   │   ├── profile/
│   │   │   └── hasCompleteProfileName.ts
│   │   ├── questions/
│   │   │   └── getQuestionsWithOptions.ts  # Full Q&A data
│   │   └── subscriptions/
│   │       ├── getFreeTierStatus.ts
│   │       ├── getModulePlans.ts
│   │       ├── getModuleSubscriptions.ts
│   │       ├── getSubscriptionStatus.ts
│   │       └── hasAnySubscription.ts
│   │
│   ├── supabase/                # Database client configuration
│   │   ├── server.ts            # Server-side Supabase client (SSR with cookies)
│   │   └── client.ts            # Browser Supabase client (localStorage)
│   │
│   ├── contracts/               # Type definitions and schemas
│   │   ├── attempts.ts          # AttemptListRow, AttemptMeta types
│   │   ├── lib/                 # Shared type utilities
│   │   └── ...
│   │
│   ├── ui/                      # UI utilities and formatting
│   │   ├── attempts.ts          # Format functions (formatDateTime, attemptScorePercent, statusLabel)
│   │   └── attemptOptionOrder.ts # Answer ordering/shuffling logic
│   │
│   ├── env.ts                   # Environment variable validation and helpers
│   ├── taxonomy.ts              # Subject/topic/subtopic hierarchies
│   └── ...
│
├── public/                      # Static assets (images, fonts, brand)
│   └── brand/                   # Logo and branding assets
│
├── scripts/                     # Operational scripts (data migration, airtable sync)
│   ├── lib/                     # Script utilities
│   └── *.mjs                    # Individual scripts
│
├── supabase/                    # Supabase configuration
│   ├── migrations/              # Database schema migrations
│   └── sql/                     # SQL helpers and RPCs
│
├── data/                        # Seed data and reference files
│   └── airtable/                # Airtable export snapshots
│
├── docs/                        # Documentation
│   └── source/                  # Documentation source files
│
├── electron/                    # Electron desktop app wrapper (optional)
│
├── .github/workflows/           # CI/CD pipeline definitions
├── .vscode/                     # VSCode settings
├── .claude/                     # Claude workspace settings
│
├── .env.example                 # Environment variables template
├── .env.local                   # Local environment (secrets - not committed)
├── .gitignore                   # Git ignore rules
├── .cursorrules                 # Cursor AI editor rules
│
├── package.json                 # Dependencies and npm scripts
├── package-lock.json            # Locked dependency versions
├── tsconfig.json                # TypeScript configuration
├── next.config.ts               # Next.js configuration
├── eslint.config.mjs            # ESLint configuration
├── tailwind.config.js           # Tailwind CSS configuration
├── postcss.config.js            # PostCSS configuration
├── electron-builder.json        # Electron build configuration
│
├── README.md                    # Project readme
└── proxy.ts                     # Development proxy utilities
```

## Directory Purposes

**`app/`**
- Purpose: Next.js App Router source
- Contains: Pages (`.tsx` files named `page.tsx` or `layout.tsx`), API routes, and React components
- Key files: `layout.tsx` (root), `error.tsx` (error boundary)

**`app/api/`**
- Purpose: API endpoints for mutations and webhooks
- Contains: HTTP POST/GET handlers with service role access
- Naming: `route.ts` for each endpoint
- Authentication: Every route checks `supabaseServer().auth.getUser()` at start

**`app/admin/`**
- Purpose: Admin-only pages
- Contains: User management, question publishing, subscription administration
- Access control: Pages check RPC `is_admin()` before rendering

**`app/attempts/[attempt_id]/`**
- Purpose: Quiz attempt interface
- Contains: Main quiz page, review page, UI components for question display and answer selection
- Key components: `AttemptQuestionsOneByOne.tsx` (carousel), `AnswerPicker.tsx` (selection logic), `TimeRemaining.tsx` (timer)

**`lib/server/`**
- Purpose: Server-only data fetching and business logic
- Decorated with `"use server"` or imported only in server components
- Organized by domain: `attempts/`, `questions/`, `subscriptions/`, `profile/`, `analytics/`
- Return types: Use result pattern (`{ ok: true; data: T }` or `{ ok: false; reason: string }`)

**`lib/api/`**
- Purpose: Browser-safe API interactions
- Files ending in `.client.ts` are client-only; others can be called from server components
- Example: `assessments.ts` uses `supabaseServer()`, while `presence.client.ts` uses browser client

**`lib/supabase/`**
- Purpose: Supabase client factory functions
- `server.ts`: Returns authenticated server client with SSR support
- `client.ts`: Returns browser client (not used in current implementation—see `lib/api/*.client.ts`)

**`lib/contracts/`**
- Purpose: Shared TypeScript type definitions
- `attempts.ts`: AttemptListRow, AttemptMeta types used across layers

**`lib/ui/`**
- Purpose: Non-domain-specific formatting and display utilities
- `attempts.ts`: Formatting functions (formatDateTime, statusLabel, attemptScorePercent)
- `attemptOptionOrder.ts`: Answer option shuffling/ordering logic

**`public/`**
- Purpose: Static assets served by Next.js
- Subdirectories: `brand/` (logos, favicons)

**`scripts/`**
- Purpose: Operational scripts for data import/export, airtable sync
- Naming: `*.mjs` (ES modules); run via `npm run script-name`
- Examples: `sync-airtable-to-supabase.mjs`, `publish-airtable-to-supabase.mjs`

**`supabase/`**
- Purpose: Database schema and configuration
- `migrations/`: SQL migration files (auto-generated by `supabase migrate` CLI)
- `sql/`: Custom SQL helpers and RPC function definitions

**`data/`**
- Purpose: Reference data and seed files
- `airtable/`: Snapshots of Airtable exports for validation or migration

**`docs/`**
- Purpose: Project documentation
- `source/`: Markdown or text documentation files

## Key File Locations

**Entry Points:**
- `app/layout.tsx`: Root layout wrapping all pages (header, footer, providers)
- `app/page.tsx`: Home page (likely redirects to dashboard if authenticated)
- `app/dashboard/page.tsx`: Main authenticated dashboard
- `app/api/attempts/start/route.ts`: Creates new attempt record
- `app/api/attempts/save/route.ts`: Persists quiz answers

**Configuration:**
- `.env.example`: Lists all required environment variables
- `.env.local`: Actual secrets (not committed)
- `next.config.ts`: Next.js build config; loads Supabase URL into `env` object
- `tsconfig.json`: TypeScript compiler options; defines path alias `@/*` for absolute imports
- `tailwind.config.js`: Tailwind CSS theme
- `eslint.config.mjs`: ESLint rules

**Core Logic:**
- `lib/server/attempts/getAttemptMeta.ts`: Loads attempt metadata, validates ownership, fetches timing rules
- `lib/server/attempts/getAttemptQuestionIds.ts`: Gets question IDs for attempt
- `lib/server/questions/getQuestionsWithOptions.ts`: Loads full Q&A data with options
- `lib/api/assessments.ts`: Lists published assessments with subscription filtering
- `lib/supabase/server.ts`: Instantiates Supabase server client with cookie-based auth

**Testing:**
- Test files co-located with source (if any) or in separate `__tests__/` directories
- None visible in current structure (may be added later)

## Naming Conventions

**Files:**
- `.tsx` for React components
- `.ts` for pure TypeScript/utilities
- `.mjs` for ES module scripts (not using CommonJS)
- `page.tsx` for Next.js page routes
- `layout.tsx` for Next.js layouts
- `route.ts` for API route handlers
- `*.client.ts` for browser-only code
- Names ending in `-Form`, `-Button`, `-Picker` for UI components

**Directories:**
- Lowercase with hyphens: `app/admin/`, `lib/server/`, `app/api/billing/`
- Dynamic routes in square brackets: `[attempt_id]`, `[id]`

**Functions & Variables:**
- camelCase for function names: `getAttemptMeta`, `formatDateTime`, `parseSavedAnswers`
- camelCase for variables and constants: `attemptId`, `serviceRoleKey`, `ALLOWED_TOPIC_NAMES`
- PascalCase for React components and type names: `AttemptQuestionsOneByOne`, `AttemptMeta`, `AssessmentListRow`
- Uppercase with underscores for constants: `ALLOWED_TOPIC_NAMES`

## Where to Add New Code

**New Feature (e.g., quiz performance insights):**
- Primary code:
  - Server function: `lib/server/analytics/getNewFeatureData.ts` (with `"server-only"` import)
  - API route: `app/api/new-feature/route.ts` (if mutation needed)
  - Page: `app/new-feature/page.tsx` or nested under existing section
- Tests: Create `lib/server/analytics/__tests__/getNewFeatureData.test.ts` (pattern TBD)

**New Component/Module (e.g., quiz timer countdown):**
- Implementation:
  - UI component: `app/attempts/[attempt_id]/CountdownTimer.tsx`
  - Utilities: `lib/ui/timer.ts` (formatting, calculation helpers)
  - Server fetches: `lib/server/attempts/getTimerRules.ts` (if needs server data)

**Utilities (e.g., new formatting function):**
- Shared helpers: `lib/ui/helpers.ts` or domain-specific like `lib/ui/attempts.ts`
- Non-UI logic: Add to domain module (e.g., `lib/server/questions/` for question utils)

**API Endpoints (e.g., new mutation):**
- Location: `app/api/[domain]/[action]/route.ts`
- Pattern: `export async function POST(req: Request) { ... }`
- Always start with auth check: `const { data: { user }, error: userErr } = await supabaseServer().auth.getUser()`

**Server Functions (e.g., new data fetch):**
- Location: `lib/server/[domain]/[functionName].ts`
- Pattern: Decorate with `"use server"` or pure function + export
- Return type: Prefer result pattern `{ ok: true; data: T } | { ok: false; reason: string }`
- Always validate inputs with type guards or regex

## Special Directories

**`.next/`**
- Purpose: Next.js build output
- Generated: Yes (by `npm run build`)
- Committed: No (in `.gitignore`)
- Contains: Compiled pages, server functions, dependency manifest

**`node_modules/`**
- Purpose: npm dependencies
- Generated: Yes (by `npm install`)
- Committed: No (in `.gitignore`)
- Contains: All third-party packages

**`.git/`**
- Purpose: Git repository metadata
- Generated: Yes (by `git init`)
- Committed: No (is git metadata)
- Contains: Commit history, branches, worktrees (for Claude workspace management)

**`.vercel/`**
- Purpose: Vercel deployment configuration
- Generated: Yes (by Vercel platform)
- Committed: No (in `.gitignore`)
- Contains: Build cache, deployment metadata

**`supabase/`**
- Purpose: Database schema and migration tracking
- Generated: Partially (migrations auto-generated)
- Committed: Yes (migrations are versioned)
- Contains: SQL migration files with timestamps

**`scripts/`**
- Purpose: Operational scripts for data management
- Generated: No (manually written)
- Committed: Yes
- Pattern: Import shared utilities from `scripts/lib/`, use `.mjs` for ES modules

---

*Structure analysis: 2026-03-22*
