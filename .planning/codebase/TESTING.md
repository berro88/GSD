# Testing Patterns

**Analysis Date:** 2026-03-22

## Test Framework

**Runner:**
- Not detected in current configuration
- No Jest, Vitest, or Mocha setup found in package.json
- No `.test.ts`, `.spec.ts`, or `.test.tsx` files in source code (outside node_modules)

**Assertion Library:**
- Node.js `assert` module used for contract testing
- Script-based testing with `import assert from "node:assert/strict"`

**Run Commands:**
```bash
npm run test:rpc-contracts              # Test Supabase RPC contracts and DB schema
npm run test:lifecycle-staging          # Test attempt lifecycle in staging environment
```

## Test File Organization

**Location:**
- No traditional unit/integration tests in TypeScript source
- Test scripts located in `scripts/` directory as `.mjs` files (Node.js CommonJS with ES modules)
- Test files: `scripts/test-rpc-contracts.mjs`, `scripts/test-attempt-lifecycle-staging.mjs`

**Naming:**
- Script files: `test-*.mjs` prefix for test/validation scripts
- Pattern: test scripts validate external integrations (Supabase, staging environment)

**Structure:**
```
scripts/
├── test-rpc-contracts.mjs           # Contract/schema validation
└── test-attempt-lifecycle-staging.mjs # End-to-end lifecycle test
```

## Test Structure

**RPC Contract Testing Pattern:**

The primary test suite validates database contracts using Node.js assert. Located in `scripts/test-rpc-contracts.mjs`:

```javascript
import dotenv from "dotenv";
import assert from "node:assert/strict";
import { createClient } from "@supabase/supabase-js";

dotenv.config({ path: ".env.local" });

// Setup: Load env vars and create Supabase clients
const URL = process.env.SUPABASE_URL || process.env.NEXT_PUBLIC_SUPABASE_URL;
const SERVICE_KEY = process.env.SUPABASE_SERVICE_ROLE_KEY;
const ANON_KEY = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY;

if (!URL || !SERVICE_KEY || !ANON_KEY) {
  throw new Error("Missing Supabase env vars...");
}

const service = createClient(URL, SERVICE_KEY, { auth: { persistSession: false } });
const anon = createClient(URL, ANON_KEY, { auth: { persistSession: false } });

// Define constants for what must exist
const REQUIRED_TABLES = ["test_attempts", "question_set_snapshots", ...];
const REQUIRED_RPCS = ["rpc_start_attempt", "rpc_save_attempt_answers", ...];

// Test functions check each invariant
async function checkOpenApiSurface() {
  // Fetch OpenAPI spec from Supabase REST API
  // Assert each required table and RPC exists
}

async function checkCoreTableReachability() {
  // Verify service role can read core tables
}

async function checkAuthBoundaries() {
  // Verify anonymous user cannot access protected RPCs
}

// Run all tests
async function main() {
  console.log("RPC contract smoke test: start");
  await checkOpenApiSurface();
  console.log("OpenAPI surface: OK");
  // ... continue checks
  console.log("RPC contract smoke test: PASS");
}

main().catch((err) => {
  console.error("RPC contract smoke test: FAIL");
  console.error(err);
  process.exit(1);
});
```

**Patterns:**
- **Setup**: Load environment variables with fallback checking
- **Assertion**: Use `assert.equal()`, `assert()` with descriptive messages
- **Error handling**: Catch errors in main function, exit with code 1 on failure
- **Cleanup**: Implicit (no state modifications in tests)

## Mocking

**Framework:**
- Not detected in traditional sense
- Node.js native APIs used (fetch, environment variables)
- Supabase clients instantiated with real endpoints (for staging/production testing)

**Patterns:**
```javascript
// Create separate clients for different auth levels
const service = createClient(URL, SERVICE_KEY, { auth: { persistSession: false } });
const anon = createClient(URL, ANON_KEY, { auth: { persistSession: false } });

// Test different permission boundaries
const anonGrade = await anon.rpc("rpc_grade_attempt", { p_attempt_id: nil });
assert(isAuthOrPermError(anonGrade.error), "Anonymous user must not be able to grade attempts");
```

**What to Test:**
- RPC existence and accessibility via OpenAPI surface
- Table existence and reachability
- Authentication boundaries (permission errors for unauthorized access)
- RPC contract signatures match expected surface

**What NOT to Test:**
- Business logic in client-side code (components, hooks)
- UI rendering or visual output
- Local development setup details

## Fixtures and Factories

**Test Data:**
- No persistent fixture files detected
- Constants defined inline in test scripts:
  ```javascript
  const REQUIRED_TABLES = [
    "test_attempts",
    "question_set_snapshots",
    "assessment_rule_snapshots",
    "assessments",
    "questions",
    "answer_options",
  ];

  const REQUIRED_RPCS = [
    "rpc_start_attempt",
    "rpc_save_attempt_answers",
    "rpc_submit_attempt",
    "rpc_expire_attempt",
    "rpc_reset_attempt",
    "rpc_get_review",
    "rpc_get_attempt_meta",
    "rpc_list_my_attempts_v1",
    "rpc_grade_attempt",
  ];
  ```

**Location:**
- Test data defined in test scripts as global constants
- Environment configuration loaded from `.env.local`

## Coverage

**Requirements:**
- Not enforced via tooling
- No coverage reporting configuration found

**View Coverage:**
- Not applicable (no test runner configured)

## Test Types

**Contract/Integration Tests:**
- **Scope**: Supabase RPC schema and auth boundaries
- **Approach**: Node.js scripts using Supabase REST API
  - Validate OpenAPI surface
  - Check table accessibility
  - Verify authentication enforcement
- **Location**: `scripts/test-rpc-contracts.mjs`
- **Run**: `npm run test:rpc-contracts`

**Lifecycle Tests:**
- **Scope**: End-to-end attempt workflow in staging
- **Approach**: Script-based testing of full lifecycle
- **Location**: `scripts/test-attempt-lifecycle-staging.mjs`
- **Run**: `npm run test:lifecycle-staging`

**Unit Tests:**
- Not found in codebase

**E2E Tests:**
- Only lifecycle staging test qualifies; appears to exercise workflow directly against staging DB

## Common Patterns

**Error Detection:**
```javascript
function isAuthOrPermError(err) {
  if (!err) return false;
  const msg = String(err.message || "").toLowerCase();
  const code = String(err.code || "");
  return (
    code === "42501" ||
    msg.includes("permission denied") ||
    msg.includes("unauthenticated")
  );
}

// Usage
const anonGrade = await anon.rpc("rpc_grade_attempt", { p_attempt_id: nil });
assert(
  isAuthOrPermError(anonGrade.error),
  "Anonymous user must not be able to grade attempts"
);
```

**Async Testing:**
- All test functions marked `async`
- `await` used for all async operations
- Main function wrapped in `.catch()` handler

```javascript
async function main() {
  console.log("RPC contract smoke test: start");
  await checkOpenApiSurface();
  console.log("OpenAPI surface: OK");
  // ... more checks
  console.log("RPC contract smoke test: PASS");
}

main().catch((err) => {
  console.error("RPC contract smoke test: FAIL");
  console.error(err);
  process.exit(1);
});
```

**Assertion Pattern:**
- Descriptive messages always provided
- Format: `assert(condition, "Human-readable failure message")`
- Assertions check invariants, not test behavior

```javascript
assert.equal(res.status, 200, "OpenAPI introspection should return 200");
assert(paths.includes(`/${table}`), `Missing table/view in API surface: ${table}`);
```

## Testing Gaps

**No coverage for:**
- React components and hooks (no unit test framework)
- API route handlers (no test utilities)
- Client-side utility functions
- Error handling in client code
- Form validation and submission flows
- Permission boundaries in client code (validated only at DB level)

**Why no unit tests:**
- Project focused on API/RPC contracts with backend validation via Supabase
- UI testing would require separate test runner (Jest/Vitest/Playwright)
- Security model relies on SECURITY DEFINER RPCs; validation happens at database level

**Recommendation for future testing:**
- Add Jest or Vitest for API route testing (handlers in `app/api/`)
- Add Playwright for critical user workflows (authentication, attempt creation)
- Mock Supabase clients for API route tests
- Keep contract tests as smoke test before deployment

---

*Testing analysis: 2026-03-22*
