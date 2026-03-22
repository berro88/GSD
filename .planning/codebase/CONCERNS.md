# Codebase Concerns

**Analysis Date:** 2026-03-22

## Tech Debt

**Large Monolithic API Route:**
- Issue: `app/api/attempts/start/route.ts` is 605 lines - contains question selection logic, topic filtering, attempt initialization, snapshot creation, and timeout calculation all in one file
- Files: `app/api/attempts/start/route.ts`
- Impact: Difficult to test individual pieces, high coupling, maintenance burden when business logic changes
- Fix approach: Extract question selection logic to `lib/server/questions/selectQuestions.ts`, topic filtering to `lib/server/topics/filterByTopics.ts`, and timeout calculation to `lib/server/attempts/calculateTimeout.ts`

**Silent Error Handling in JSON Parsing:**
- Issue: Multiple routes catch JSON parse errors and silently return empty objects or null without distinguishing between invalid JSON and network errors
- Files: `app/api/billing/paystack/initialize/route.ts` (line 22), `app/api/attempts/start/route.ts` (line 172), `app/api/attempts/save/route.ts`, `app/admin/questions/PublishQuestionForm.tsx`
- Impact: Clients receive `400 Bad Request` without knowing if payload was malformed or parsing failed; debugging becomes harder
- Fix approach: Create `lib/server/parsing/safeParse.ts` that explicitly logs parse failures and returns typed errors with details

**Missing Request Validation Schema:**
- Issue: API routes validate individual fields inline with manual type checks instead of using schema validation (no Zod, no validation libraries)
- Files: All routes in `app/api/*/route.ts`
- Impact: Field validation is inconsistent, validation errors don't have standard format, adding fields is error-prone
- Fix approach: Adopt Zod (already in node_modules via zod dependency) for request validation in all routes

**Fragile Admin Authorization Pattern:**
- Issue: Admin checks use custom `normalizeAdminResult()` that tries to parse RPC response in multiple formats (boolean, array with object). If RPC format changes, multiple routes break
- Files: `app/api/admin/attempts/[attempt_id]/grade/route.ts` (line 9), `app/api/admin/subscription/add-module/route.ts` (line 11)
- Impact: Authorization logic scattered across routes; if backend RPC response format changes, must update all routes; inconsistent type narrowing
- Fix approach: Create `lib/server/auth/getAdminStatus.ts` that returns a single strongly-typed response; use that everywhere

## Known Bugs

**Potential Race Condition in Paystack Webhook:**
- Symptoms: If two webhook events arrive for same user/assessment nearly simultaneously, upsert operations might conflict or miss state updates
- Files: `app/api/billing/paystack-webhook/route.ts` (lines 95-116)
- Trigger: Fast webhook delivery from Paystack for subscription state changes while user is actively interacting
- Workaround: Current upsert handles conflicts via `onConflict`, but concurrent updates to different columns (status, period_end) could result in partial updates being lost

**Silent Suppression of Supabase Insert Errors:**
- Symptoms: Paystack checkout initialization silently fails to track the transaction if Supabase insert fails, but user still gets redirected to payment
- Files: `app/api/billing/paystack/initialize/route.ts` (lines 148-161)
- Trigger: Supabase down or network error during insert
- Workaround: None - webhook sync will recover eventually but transient tracking is lost

**Question Set Snapshot Polling with Fixed Retry Limit:**
- Symptoms: If question_set_snapshot_id is not populated within 600ms (8 retries × 75ms), attempt fails despite snapshot being created asynchronously
- Files: `app/api/attempts/start/route.ts` (lines 232-246)
- Trigger: Database lag or slow initial snapshot insertion
- Workaround: Increasing retry count/wait time helps but doesn't address root cause

## Security Considerations

**Anon Key Exposed in Client Config:**
- Risk: Supabase anon key is loaded into browser via `NEXT_PUBLIC_SUPABASE_ANON_KEY` and client-side code creates browser client with it. While Supabase is designed for this, JWT token leakage or key rotation becomes critical
- Files: `lib/supabase/client.ts`, `lib/env.ts`, `next.config.ts`
- Current mitigation: Supabase's RLS policies restrict data access at row level; anon key only allows authenticated operations via Supabase auth
- Recommendations: Implement request rate limiting at API gateway level; monitor anon key usage for suspicious patterns; consider token rotation strategy if key is ever suspected compromised

**Admin Status Check Relies on RPC Only:**
- Risk: Admin check only validates via RPC - if RPC is down or returns unexpected format, `normalizeAdminResult()` returns false, blocking all admin operations. No fallback to cache or secondary validation
- Files: `app/api/admin/*/route.ts` (all admin routes call `supabase.rpc("is_admin")`)
- Current mitigation: Supabase RPC is generally reliable; missing auth header will return 401
- Recommendations: Add caching of admin status for 60s to survive brief RPC failures; add circuit breaker pattern if RPC fails consistently

**Email-Based User Lookup Without Rate Limiting:**
- Risk: `app/api/admin/subscription/add-module/route.ts` (line 68-70) uses `admin.listUsers({perPage: 1000})` with no pagination to find user by email. Could be slow or DOS vector if admin endpoint is abused
- Files: `app/api/admin/subscription/add-module/route.ts`
- Current mitigation: Requires admin auth which is behind RPC check
- Recommendations: Replace list-all pattern with direct user lookup API if Supabase provides one; add request timeout; limit results to 100 at a time with pagination

**Webhook Signature Verification Uses timing-safe Compare:**
- Risk: Paystack webhook signature verification implements timing-safe HMAC comparison (good), but if webhook secret is ever leaked, attacker can forge valid signatures
- Files: `app/api/billing/paystack-webhook/route.ts` (lines 6-19)
- Current mitigation: HMAC-SHA512 with timing-safe comparison; secret stored in `SUPABASE_SECRET_KEY` env var
- Recommendations: Implement webhook signature rotation policy; log all webhook rejections for monitoring; consider Paystack webhook signing v2 if available

## Performance Bottlenecks

**Question Selection Performs Multiple Database Queries:**
- Problem: Selecting questions for an attempt fires 2-3 separate queries: topics list, then questions query per topic type (root names vs subtopic names)
- Files: `app/api/attempts/start/route.ts` (lines 315-477)
- Cause: Topic hierarchy requires querying all topics first, then using that to filter question queries
- Improvement path: Pre-compute topic-to-questions mapping in database view; use single `in()` query with OR logic instead of sequential queries

**Polling for Question Snapshot with Fixed Delays:**
- Problem: Creates attempts by calling RPC then polling database every 75ms for up to 600ms while waiting for snapshot to be created
- Files: `app/api/attempts/start/route.ts` (lines 232-246)
- Cause: Snapshot creation is asynchronous in RPC, client-side code must wait for it to be written
- Improvement path: Return snapshot ID from RPC immediately or use websocket subscription to wait for snapshot; eliminate polling entirely

**Paystack Plan Lookup on Every Subscription Attempt:**
- Problem: `app/api/billing/paystack/initialize/route.ts` queries module_plans table for every subscription initialization without caching
- Files: `app/api/billing/paystack/initialize/route.ts` (lines 60-65)
- Cause: Plan code and amount needed per request
- Improvement path: Cache module plans in-memory with 5-10 minute TTL; invalidate cache on admin update

**No Caching of Authentication State:**
- Problem: Every API route calls `supabaseServer().auth.getUser()` which validates the auth cookie every time
- Files: All `app/api/*/route.ts` files
- Cause: Cannot cache auth state due to per-request cookie format
- Improvement path: Use auth middleware layer to cache user lookup for duration of request; store in request context

## Fragile Areas

**Attempt Lifecycle with Multiple State Transitions:**
- Files: `app/api/attempts/start/route.ts`, `app/api/attempts/save/route.ts`, `app/attempts/[attempt_id]/page.tsx`, `lib/server/attempts/getReview.ts`
- Why fragile: Attempt creation, question assignment, answer saving, and review all modify overlapping state (test_attempts table, question_set_snapshots, answers). Missing transaction coordination or optimistic locking could lead to inconsistent state
- Safe modification: Add test coverage for state transitions (abandon during load, save after timeout, etc.); wrap multi-step operations in database transactions
- Test coverage: No automated tests for attempt lifecycle - only manual/staging tests referenced in `package.json` scripts

**Topic Hierarchy Filtering Logic:**
- Files: `app/api/attempts/start/route.ts` (lines 132-389)
- Why fragile: Complex logic to determine when to include parent topic in question pool vs only children; logic assumes parent_topic_id is reliably set
- Safe modification: Add unit tests for `collectDescendantTopicIds()` with edge cases (orphaned topics, circular references, null parents)
- Test coverage: None - untested helper functions

**Paystack Status Mapping:**
- Files: `app/api/billing/paystack-webhook/route.ts` (line 200)
- Why fragile: Maps Paystack status strings to database values with inline logic: `activeStatuses.includes(status) ? status : status === "cancelled" || status === "completed" ? "cancelled" : status`
- Safe modification: Create status enum mapping file; add tests for all Paystack statuses; log unmapped statuses for monitoring
- Test coverage: None

**Error Handling in Webhook Event Processing:**
- Files: `app/api/billing/paystack-webhook/route.ts` (lines 87-222)
- Why fragile: `upsertLegacySubscription()` and `upsertModuleSubscription()` async calls are not awaited, errors silently fail
- Safe modification: Add error handling around upsert calls; return 202 Accepted only after confirming upsert succeeded; log upsert failures
- Test coverage: None

## Scaling Limits

**User Listing Without Pagination:**
- Current capacity: ~1,000 users per call (hardcoded `perPage: 1000`)
- Limit: Will be slow or fail if organization grows beyond 10k users
- Scaling path: Implement paginated user lookup; use email index directly if Supabase admin API supports it

**Question Pool Selection Complexity:**
- Current capacity: ~2,000 published questions per assessment
- Limit: Multiple topic-based queries scale poorly; at 5k+ questions, query times become significant
- Scaling path: Pre-compute question pools in database; use full-text search or indexing for tags

**Concurrent Attempt Starts:**
- Current capacity: Database RPC and snapshot creation can handle ~100 concurrent starts per second
- Limit: At higher concurrency (exams with 1000+ simultaneous test-takers), RPC queue will become bottleneck
- Scaling path: Implement attempt queue with async processing; use database queue table; add read replicas for question lookups

## Dependencies at Risk

**Next.js 16.1.6:**
- Risk: Using a newer major version of Next.js (v16); minor updates could introduce breaking changes to middleware, streaming, or server component behavior
- Impact: Updates to Next.js could break SSR auth flow, cookie handling, or API route responses
- Migration plan: Pin to specific minor version; test all auth flows before updating to next major version

**Supabase SSR 0.8.0:**
- Risk: SSR cookie handling is critical for auth flow; if Supabase updates cookie format or auth token structure, could break all authenticated operations
- Impact: Users would be logged out; API calls would fail with 401 errors
- Migration plan: Review Supabase changelogs before dependency updates; test auth flow in staging after upgrades

**React 19.2.3:**
- Risk: Using React 19 (major update from 18); use of useOptimistic or other new hooks could introduce subtle state management bugs
- Impact: Components using new features could have stale state or race conditions
- Migration plan: Avoid new React 19 features unless thoroughly tested; gradually migrate components

## Missing Critical Features

**No Offline Support:**
- Problem: User cannot start attempt or save answers if network fails mid-quiz
- Blocks: Handling unreliable network in exam environments
- Workaround: User must abandon and restart attempt

**No Audit Logging:**
- Problem: No log of who modified what (e.g., admin grading, subscription changes) - only database transaction logs exist
- Blocks: Compliance requirements, fraud investigation, debugging permission issues
- Workaround: Manual database queries to find transaction timestamps

**No Request Rate Limiting:**
- Problem: No rate limiting on API endpoints - malicious user could spam attempts/saves
- Blocks: Protection against DOS and abuse
- Workaround: Rely on infrastructure-level rate limiting (Vercel/CDN)

**No Graceful Degradation for Paystack Failures:**
- Problem: If Paystack is down, users cannot subscribe - no fallback payment method
- Blocks: Revenue during Paystack outages
- Workaround: Manual admin override via add-module endpoint (slow, requires admin)

## Test Coverage Gaps

**No Unit Tests for API Route Logic:**
- What's not tested: All parsing functions (parseQuestionLimit, parseRequestedCount, etc.), topic filtering, admin checks, webhook signature verification
- Files: `app/api/attempts/start/route.ts`, `app/api/admin/*/route.ts`, `app/api/billing/paystack-webhook/route.ts`
- Risk: Business logic changes break silently; bugs in parsing logic go undetected
- Priority: High - affects user-facing functionality and billing

**No Integration Tests for Attempt Lifecycle:**
- What's not tested: Creating attempt → assigning questions → saving answer → timeout expiry → review flow
- Files: `app/api/attempts/start/route.ts`, `app/api/attempts/save/route.ts`, `app/attempts/[attempt_id]/page.tsx`, `lib/server/attempts/getReview.ts`
- Risk: State transitions could break in subtle ways (e.g., attempt marked complete but questions still assignable)
- Priority: High - core product feature

**No Tests for Subscription State Transitions:**
- What's not tested: Subscription creation → webhook processing → active/cancelled status
- Files: `app/api/billing/paystack-webhook/route.ts`, `lib/server/subscriptions/getModuleSubscriptions.ts`
- Risk: Users could lose access or gain unauthorized access if subscription state machine breaks
- Priority: Critical - affects revenue and access control

**No E2E Tests:**
- What's not tested: Full user flows (register → subscribe → take assessment → review)
- Files: All `app/` routes and components
- Risk: Integration issues between frontend and API only caught in manual testing
- Priority: Medium - handled by staging environment tests but no CI coverage

**Client-Side Async Error Handling Not Tested:**
- What's not tested: Network failures, timeout handling, race conditions in answer saving
- Files: `lib/api/attempts.client.ts`, components using `saveAttemptAnswers()`
- Risk: Silent failures if network drops; users don't know if answer was saved
- Priority: Medium

---

*Concerns audit: 2026-03-22*
