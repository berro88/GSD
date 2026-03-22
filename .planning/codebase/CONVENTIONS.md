# Coding Conventions

**Analysis Date:** 2026-03-22

## Naming Patterns

**Files:**
- Components: PascalCase (e.g., `PublishQuestionForm.tsx`, `LogoutButton.tsx`, `PresencePinger.tsx`)
- API routes: kebab-case in directories (e.g., `/api/admin/questions/publish`, `/api/attempts/start`)
- Utilities and helpers: camelCase (e.g., `hasCompleteProfileName.ts`, `getAttempt.ts`)
- Type/contract files: camelCase (e.g., `attempts.ts`, `assessments.ts`)

**Functions:**
- camelCase for all functions (e.g., `saveAttemptAnswers()`, `pingPresence()`, `parseQuestionLimit()`, `normalizeAdminResult()`)
- Helper/utility functions: descriptive, prefixed with verb or adjective (e.g., `collectDescendantTopicIds()`, `extractAnsweredQuestionIds()`, `requireEnv()`)
- Async functions clearly marked with `async` keyword
- Server-side utilities use descriptive names like `hasAccessToModule()`, `getFreeTierStatus()`

**Variables:**
- camelCase for all variables and constants
- Constants in uppercase with underscores (e.g., `HEARTBEAT_MS = 30_000`, `FREE_TIER_MAX_PER_ATTEMPT = 25`, `SECONDS_PER_QUESTION_DEFAULT = 72`)
- Underscore-prefixed for unused parameters in error handling (e.g., `catch () {}` to ignore errors silently when appropriate)
- Type-narrowing suffixes: `*Err` for errors (e.g., `myAttemptErr`, `userErr`, `bpErr`), `*Data` for response data (e.g., `adminData`, `latestBlueprint`)
- Ref variables suffixed with `Ref` (e.g., `lastPathRef`)

**Types:**
- PascalCase for all types and interfaces (e.g., `AttemptListRow`, `BlueprintShape`, `TopicRow`, `StartBody`)
- Domain-specific suffixes: `*Row` for database rows, `*Shape` for partial/validation shapes
- Discriminated union patterns for result types (e.g., `{ ok: true } | { ok: false; error: string }`)

## Code Style

**Formatting:**
- ESLint configuration: `eslint.config.mjs` (flat config format)
- Next.js ESLint config used as base: `eslint-config-next/core-web-vitals` and `eslint-config-next/typescript`
- Indentation: 2 spaces (inferred from code samples)
- Line length: No enforced limit, but code is readable and breaks at reasonable points

**Linting:**
- Tool: ESLint 9+
- Key rules:
  - `@typescript-eslint/no-unused-vars`: Warns on unused variables, except those prefixed with `_`
  - Pattern ignores arguments and variables starting with underscore for intentionally unused values

**TypeScript:**
- Strict mode enabled in `tsconfig.json`
- Target: ES2017
- No `null` return patterns—use explicit types
- Type guards used for validation (e.g., `if (typeof data === "boolean")`, `if (Array.isArray(data))`)

## Import Organization

**Order:**
1. Standard library imports (`path`, `fs`)
2. Third-party frameworks/libraries (`next/*`, `react`, `@supabase/*`)
3. Internal utilities and helpers (`@/lib/*`)
4. Local imports (relative paths or same-directory imports)
5. Type imports (marked with `type` keyword)

**Path Aliases:**
- Configured in `tsconfig.json`: `@/*` → `./*` (project root)
- All internal imports use `@/` prefix (e.g., `@/lib/env`, `@/lib/server/profile/hasCompleteProfileName`)
- Enables consistency across file structure changes

**Example:**
```typescript
import type { NextConfig } from "next";
import path from "path";
import fs from "fs";
import { supabaseServer } from "@/lib/supabase/server";
import { hasAccessToModule } from "@/lib/server/subscriptions/getModuleSubscriptions";
import PublishQuestionForm from "./PublishQuestionForm";
```

## Error Handling

**Patterns:**
- Return-based error handling for client operations: `Promise<{ ok: true } | { ok: false; error: string }>`
  - Example: `saveAttemptAnswers()` returns discriminated union indicating success/failure with error message
- Throw-based error handling for server operations: `throw new Error(error.message)`
  - Used in helper functions like `listPublishedAssessments()` and environment loaders
- Validation errors handled with try-catch and detailed user messaging
- Silent failures for non-critical operations (e.g., presence ping): `catch {}` blocks without logging
  - Comment explaining why: "Presence must not block core user flows"
- JSON parsing with fallback: `await res.json().catch(() => null)` returns null on parse failure
- Type narrowing for safety: Cast unknown response types with `as` only after type guards

**Example - Return-based:**
```typescript
export async function saveAttemptAnswers(
  attemptId: string,
  answers: Record<string, string | null>
): Promise<{ ok: true } | { ok: false; error: string }> {
  const res = await fetch("/api/attempts/save", { ... });
  const body = (await res.json().catch(() => null)) as { ok?: boolean; error?: string } | null;
  if (!res.ok) return { ok: false, error: body?.error || "save_failed" };
  if (!body || body.ok !== true) return { ok: false, error: "unknown_error" };
  return { ok: true };
}
```

**Example - Throw-based:**
```typescript
function requireEnv(name: string): string {
  const value = process.env[name];
  const str = typeof value === "string" ? value : "";
  if (!str || str.trim() === "") {
    throw new Error(`Missing ${name}. Add it to .env.local (see README for required vars).`);
  }
  return str.trim();
}
```

## Logging

**Framework:** `console` (implicit, no explicit logging library detected)

**Patterns:**
- No logging calls found in business logic or API routes
- Error messages passed through response bodies with descriptive error codes
- Errors formatted as: `"error": "message with reason"` in JSON responses
- Detailed error information in error responses: `{ error: string, detail?: string, code?: string | null }`

**Example:**
```typescript
if (userErr || !user) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}

if (myAttemptErr || !myAttempt?.question_set_snapshot_id) {
  return NextResponse.json({
    error: `Unable to load created attempt snapshot (attempt_id=${attemptId})`,
    detail: myAttemptErr?.message ?? null,
  }, { status: 400 });
}
```

## Comments

**When to Comment:**
- Domain-specific logic and business rules: Comments explain WHY, not WHAT
- Complex algorithms: Explain approach before implementation
- Non-obvious type narrowing: Clarify why `as` cast is safe
- Architecture decisions: Explain immutability constraints and RPC discipline

**Example:**
```typescript
// Stage restriction: only these topic/subtopic names should be available via
// `/assessments` at this time.
const ALLOWED_TOPIC_NAMES = [ ... ];

// Candidate question pool selection
//
// Important: for subtopic drill-down we filter by `questions.topic_name`
// (12 major topics) and `questions.subtopic_name` (Airtable subtopics),
// not by `questions.topic_id`, because topic_id is not reliably keyed to
// the same granularity as the UI selection.
```

**JSDoc/TSDoc:**
- Not extensively used in source code
- Type signatures are explicit in function declarations
- Large data flow comments appear above complex sections (e.g., question selection logic)

## Function Design

**Size:** Functions range from 5-600 lines, with complex API routes being larger
- Client wrappers: 10-25 lines
- Simple utilities: 5-15 lines
- API routes: 100-600 lines (complex business logic, database operations)
- Server helpers: 30-80 lines (data fetching with error handling)

**Parameters:**
- Use typed objects for functions with multiple parameters
- Example: `StartBody` type with optional fields for `/api/attempts/start`
- Destructuring used in async functions for clarity
- Type annotations always explicit

**Return Values:**
- Discriminated unions for client operations (ok/error variants)
- Thrown errors for server operations
- Explicit return types in function signatures
- Async functions always return `Promise<T>`
- Never return `undefined` implicitly—use explicit `null` or specific return type

**Example:**
```typescript
function parseQuestionLimit(blueprint: BlueprintShape | null): number {
  const raw = blueprint?.n_questions;
  const n = typeof raw === "number" ? raw : typeof raw === "string" ? Number(raw) : NaN;
  if (Number.isFinite(n) && n >= 1) {
    return Math.min(200, Math.floor(n));
  }
  return 20;
}
```

## Module Design

**Exports:**
- Named exports for utilities: `export function`, `export type`
- Default exports for React components and page routes
- Clear separation: components export default, utilities export named

**Barrel Files:**
- Not detected in codebase
- Direct imports used (e.g., `from "@/lib/env"`)

**File Organization:**
- One primary export per file
- Related types exported alongside functions
- Type definitions at top of file before implementations
- API route files: type definitions → helper functions → main handler export

**Example:**
```typescript
// lib/contracts/attempts.ts - Types only
export type AttemptListRow = { ... };

// lib/env.ts - Utilities with named exports
export function getSupabasePublicEnv() { ... }
export function getSupabaseServiceEnv() { ... }

// app/admin/questions/page.tsx - Default export for page
export default async function AdminQuestionsPage() { ... }
```

## Client vs Server Code

**Client Components:**
- Marked with `"use client"` directive at top
- Use React hooks: `useState`, `useEffect`, `useRef`, `useMemo`
- Use Next.js client navigation: `useRouter`, `usePathname`
- Import from `@/lib/api/*` for client-side API wrappers

**Server Components:**
- No directive (default in App Router)
- `async` component functions
- Direct database access via `supabaseServer()`
- Use `redirect()` for authentication checks

**Example - Server:**
```typescript
// app/page.tsx - async server component
export default async function HomePage() {
  const supabase = await supabaseServer();
  const { data } = await supabase.auth.getUser();
  if (!data.user) redirect("/login");
  ...
}
```

**Example - Client:**
```typescript
// app/LogoutButton.tsx - client component
"use client";
export default function LogoutButton() {
  const [loading, setLoading] = useState(false);
  const router = useRouter();
  async function onLogout() { ... }
  ...
}
```

## Architecture Rules (from .cursorrules)

**Non-negotiable:**
1. **Snapshot Immutability**: Never update snapshot rows after creation
2. **Attempt Lifecycle**: Forward-only transitions (started → submitted → graded → reviewed)
3. **RPC Discipline**: Use SECURITY DEFINER RPCs for lifecycle changes, never bypass with direct writes
4. **Reveal Gating**: UI relies on DB truth; if `correct_option_id` is NULL, nothing is revealed
5. **Minimal Diff Policy**: Make smallest possible change, no refactors unless requested
6. **Schema Protection**: Preserve RLS assumptions, always use `auth.uid()` for user scoping
7. **Security First**: No secret keys in code, never expose `service_role` keys to client

---

*Convention analysis: 2026-03-22*
