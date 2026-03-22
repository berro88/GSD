# Architecture

**Analysis Date:** 2026-03-22

## Pattern Overview

**Overall:** Full-stack Next.js 16 monolith with server-side rendering, API routes, and real-time Supabase integration

**Key Characteristics:**
- Server-centric data fetching with React Server Components (RSC)
- API route handlers for mutation operations and webhooks
- Supabase as primary data persistence and authentication backend
- Layered separation: UI pages → API routes → Server functions → Supabase
- Real-time presence and event tracking for quiz attempt sessions

## Layers

**Page/UI Layer:**
- Purpose: Render Next.js pages and interactive React components
- Location: `app/` directory (Next.js App Router)
- Contains: Page components (`page.tsx`), route-specific client components, forms, and UI
- Depends on: Server functions, API routes, client-side utilities
- Used by: Browsers, Electron client

**API Route Layer:**
- Purpose: Handle HTTP mutations, webhooks, and business logic requiring service role credentials
- Location: `app/api/` and subdirectories
- Contains: POST/GET/PUT handlers using service role Supabase client
- Depends on: Server functions, environment variables, Supabase service role key
- Used by: Pages, client-side fetch calls, external webhooks (Paystack)
- Examples: `app/api/attempts/save/`, `app/api/auth/`, `app/api/billing/`

**Server Functions Layer:**
- Purpose: Execute server-only logic with RPC calls and complex queries
- Location: `lib/server/` organized by domain (attempts, questions, subscriptions, profile, analytics)
- Contains: Functions decorated with `"server-only"` directive, RPC wrappers, data transformation
- Depends on: Supabase server client, type contracts
- Used by: Pages, API routes
- Examples: `lib/server/attempts/getAttemptMeta.ts`, `lib/server/subscriptions/getSubscriptionStatus.ts`

**Client API Layer:**
- Purpose: Browser-side data fetching and client-only operations
- Location: `lib/api/` for client-safe fetches and `lib/api/` files ending in `.client.ts`
- Contains: Fetch wrappers for public endpoints, client Supabase methods
- Depends on: Browser Supabase client, fetch API
- Used by: Client components, utilities
- Examples: `lib/api/attempts.client.ts`, `lib/api/presence.client.ts`

**Data Access Layer:**
- Purpose: Instantiate and configure Supabase clients (server and browser)
- Location: `lib/supabase/`
- Contains: Server and browser client factories
- Depends on: Environment variables, Supabase SDK
- Used by: All server functions, API routes, client code
- Files: `lib/supabase/server.ts`, `lib/supabase/client.ts`

**Configuration & Contracts:**
- Purpose: Type definitions, environment configuration, shared types
- Location: `lib/contracts/`, `lib/env.ts`, `lib/taxonomy.ts`
- Contains: TypeScript type exports (AttemptListRow), environment variable schema, taxonomy lookups
- Used by: All layers
- Examples: `lib/contracts/attempts.ts` (AttemptListRow), `lib/env.ts` (getSupabasePublicEnv)

**Utility Layer:**
- Purpose: Shared helpers for formatting, ordering, and UI logic
- Location: `lib/ui/`
- Contains: Non-domain-specific utilities (attemptOptionOrder, formatDateTime, statusLabel)
- Used by: Pages and components
- Examples: `lib/ui/attempts.ts`, `lib/ui/attemptOptionOrder.ts`

## Data Flow

**User Quiz Attempt (Read Path):**

1. User navigates to `/attempts/[attempt_id]` page
2. Page component calls `getAttemptMeta(attempt_id)` from `lib/server/attempts/`
3. Server function executes RPC `rpc_get_attempt_meta` on Supabase
4. Server function fetches associated rule snapshot metadata
5. Page calls `getAttemptQuestionIds()` to fetch question IDs for this attempt
6. Page calls `getQuestionsWithOptions()` to load full question + answer option data
7. Page renders questions and accepts client-side answers via component state
8. On save/submit, client component calls `POST /api/attempts/save` with answers JSON

**User Quiz Attempt (Mutation Path):**

1. Client component calls `POST /api/attempts/save` with `{attempt_id, answers}`
2. API route handler validates user identity via `supabaseServer().auth.getUser()`
3. Handler validates attempt ownership and status using service role client
4. Handler parses and normalizes answers object (handles backward compatibility)
5. Handler updates `test_attempts.answers` column with merged answers
6. Returns 200 with success or error status

**Quiz Submission & Grading:**

1. User submits attempt via UI button
2. Client calls `POST /api/attempts/submit` with attempt_id
3. API validates submission eligibility (status=started, not expired)
4. Updates `test_attempts.status` to "submitted" and sets submitted_at timestamp
5. Supabase RPC or manual grading job scores the attempt
6. Admin can view graded attempts in `/admin/attempts/[attempt_id]/`

**Subscription & Access Control:**

1. User visits `/assessments` page (public assessment catalog)
2. Page calls `listPublishedAssessments()` from `lib/api/assessments.ts`
3. Page also calls `getSubscriptionStatus()` from `lib/server/subscriptions/`
4. Page filters visible assessments by user's module subscriptions
5. User clicks "Start Attempt" → `POST /api/attempts/start` creates new test_attempt record
6. System checks subscription status before allowing free vs. paid content

**Authentication & Presence:**

1. User logs in via Supabase Auth (handled by Auth pages)
2. Supabase session stored in HTTP-only cookie (via SSR)
3. Every page call to `supabaseServer()` automatically includes auth session
4. `PresencePinger` component periodically updates user presence in Supabase Realtime
5. Admin pages use RPC `is_admin()` to check role before rendering

**State Management:**

- **Server state (source of truth):** Supabase database; fetched fresh on each page load
- **Mutation state:** Form state in client components; optimistic updates optional
- **Session state:** Supabase Auth cookie (automatically managed)
- **Real-time state:** Presence and event streams via Supabase Realtime (not heavily used yet)
- **Attempt session state:** Answers stored in-memory in React component state; periodically flushed to `/api/attempts/save`

## Key Abstractions

**AttemptMeta:**
- Purpose: Represents complete attempt metadata including timeline, scores, and snapshots
- Examples: `lib/server/attempts/getAttemptMeta.ts`, `lib/contracts/attempts.ts`
- Pattern: Result type pattern (`{ ok: true, attempt: AttemptMeta, ruleSnapshot: RuleSnapshotMeta | null }` or error state)
- Used to: Display attempt details, check expiration, validate submission readiness

**AttemptListRow:**
- Purpose: Denormalized view of attempt for listing in dashboard/history
- Examples: `lib/contracts/attempts.ts`
- Pattern: Type-only export; data comes from Supabase view or computed query
- Includes: Computed fields like `correctness_ratio`, `unanswered_rate`

**AssessmentListRow & TopicTreeNode:**
- Purpose: Type-safe representation of published assessments and taxonomy
- Examples: `lib/api/assessments.ts`
- Pattern: Client-side function wraps Supabase query, returns typed rows
- Access control: Only published assessments shown; taxonomy filtered by ALLOWED_TOPIC_NAMES

**RuleSnapshot:**
- Purpose: Immutable capture of assessment rules at attempt creation time
- Contains: Duration limits, grading rules, question ordering
- Pattern: Created once at attempt start; referenced by attempt via foreign key

**QuestionSnapshotSet:**
- Purpose: Immutable set of questions and options for grading consistency
- Pattern: Questions and options are snapshotted; database sources remain mutable for admins

## Entry Points

**`app/layout.tsx`:**
- Location: `app/layout.tsx`
- Triggers: Every page render (root layout)
- Responsibilities: Global styles, fonts, header/footer navigation, Vercel Speed Insights, session-aware navigation

**`app/page.tsx`:**
- Location: `app/page.tsx`
- Triggers: GET `/`
- Responsibilities: Home/landing page or redirect to dashboard if authenticated

**`app/dashboard/page.tsx`:**
- Location: `app/dashboard/page.tsx`
- Triggers: GET `/dashboard`
- Responsibilities: Fetch user profile, list attempts, show analytics, enforce complete profile validation

**`app/assessments/page.tsx`:**
- Location: `app/assessments/page.tsx`
- Triggers: GET `/assessments`
- Responsibilities: List published assessments, filter by subscription, show form to start new attempt

**`app/attempts/[attempt_id]/page.tsx`:**
- Location: `app/attempts/[attempt_id]/page.tsx`
- Triggers: GET `/attempts/:id`
- Responsibilities: Load attempt metadata, fetch questions, render quiz UI, manage answer state

**`app/api/attempts/save/route.ts`:**
- Location: `app/api/attempts/save/route.ts`
- Triggers: POST `/api/attempts/save`
- Responsibilities: Validate user ownership, parse answers, merge with stored answers, update database

**`app/api/attempts/start/route.ts`:**
- Location: `app/api/attempts/start/route.ts`
- Triggers: POST `/api/attempts/start`
- Responsibilities: Create new attempt record, snapshot questions/rules, initialize answers, return attempt ID

**`app/api/auth/status/route.ts`:**
- Location: `app/api/auth/status/route.ts`
- Triggers: GET `/api/auth/status`
- Responsibilities: Return authenticated user ID or unauthenticated status (used by client to check auth)

**`app/api/billing/paystack/webhook/route.ts`:**
- Location: `app/api/billing/paystack-webhook/route.ts`
- Triggers: POST webhook from Paystack payment processor
- Responsibilities: Validate webhook signature, update subscription status, grant module access

## Error Handling

**Strategy:** Multi-layered error propagation with user-friendly fallbacks

**Patterns:**

- **Result types:** Server functions return discriminated union types (`{ ok: true; data: T } | { ok: false; reason: string; error?: Error }`) instead of throwing
  - Example: `GetAttemptMetaResult` in `lib/server/attempts/getAttemptMeta.ts`

- **API error responses:** Routes return `NextResponse.json({ error: "..." }, { status: 4xx|5xx })`
  - Validation errors: 400
  - Authorization: 401
  - Not found: 404
  - Server errors: 500

- **Page-level error boundary:** `app/error.tsx` catches rendering errors, displays friendly message with retry button
  - Special handling for missing environment variables (detects `NEXT_PUBLIC_SUPABASE` in message)

- **Client-side error handling:** Components catch Promise rejections, display UI notifications (if toast library used)

- **Data parsing errors:** Functions like `parseAttemptAnswers`, `parseSavedAnswers` use defensive typing with null checks and fallbacks to `{}` on parse failure

## Cross-Cutting Concerns

**Logging:**
- `console.error()` and `process.stdout.write()` in server functions for debugging
- Example: `getAttemptMeta` logs attempt_id and rule_snapshot_id
- Client components log errors to console only

**Validation:**
- Type-based validation: TypeScript strict mode catches structural issues
- Runtime validation: Custom parsing functions (e.g., `parseAttemptAnswers`, `normalizeAdminResult`) with defensive null checks
- Business logic validation: API routes check user_id, attempt ownership, status transitions
- Schema validation: Supabase RPC calls validate parameters server-side

**Authentication:**
- Every API route starts with `supabaseServer().auth.getUser()`; rejects 401 if no user or error
- Pages redirect to `/login` if not authenticated (e.g., `if (!user) redirect("/login")`)
- Service role client used only for privileged operations (admin actions, grading)

**Authorization:**
- Pages check RPC `is_admin()` for admin UI gates
- Attempt updates include `user_id` equality check to prevent unauthorized writes
- API routes verify attempt belongs to authenticated user before mutations

---

*Architecture analysis: 2026-03-22*
