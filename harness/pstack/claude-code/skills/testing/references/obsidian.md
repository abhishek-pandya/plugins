# Testing in Obsidian

Adapted from the user-provided "Effective testing at Vanta" guide, published March 29, 2023. This is a decision guide, not a snapshot of current APIs. Read the checkout's `AGENTS.md`, `docs/backend/testing-best-practices.md` when present, and package-specific guidance before choosing tools. Verify utility signatures and imports in the checkout. The original guide's Mocha commands, paths, and timing estimates may be outdated.

## Choose dependencies deliberately

Use factories instead of shared fixtures. Find existing `test-factories/` and `*.testutil.ts` helpers before creating another helper. Override IDs, relationships, dates, and values relevant to the expectation. Keep data deterministic even when factories generate unrelated fields.

Use `useDB()` when the test needs to validate database behavior, or the repository/model layer cannot reasonably be replaced. Do not add it to pure computation tests or use it merely to avoid designing a smaller test. Classify tests with a real database as integration tests even if the command calls them unit tests.

Prefer explicit dependencies and in-memory clients for business logic. For integration tests, preserve the real internal components whose connection the test is meant to prove. Mock third-party calls at the supported boundary. Use the repository's cleanup helpers, such as `useSandbox()`, when stubbing is necessary. Do not stub the system under test or assert private helper calls.

Use current repository utilities for request objects, GraphQL resolve info, feature flags, IDs, and time. Avoid initializing real services when a provided test helper handles setup. Restore mocked clocks after each case. If a legacy Mongoose model must initialize before a mocked Date, load it before the clock override.

## Cover the system's contract

- **GraphQL resolvers.** Prefer testing the underlying business logic and calling resolvers directly with supported request and resolve-info helpers. Execute GraphQL through `getTestServer` when schema execution, authentication context, or request wiring is the behavior being tested.
- **Resource fetchers.** Test new fetchers with a fake API client and assert transformed resource output, including relevant missing-data and error cases. Use API schema validators where available. Shared credential-loading helpers belong in their own tests. If external calls cannot reasonably be isolated, test the business transformation separately; explain any omitted coverage.
- **VantaTests, automated monitors.** Test every new monitor through the full monitor flow, using `metaTestHarness` where available. Assert outcomes for relevant inputs. Helper-only tests do not prove the monitor's result.
- **Migrations.** Follow the current migration test setup. Assert the resulting data and consider schema validation after migration. Use the actual database handle required by the migration API.
- **Frontend.** Read the current frontend testing guidance and use the package's supported component or browser tools. The source document links to separate frontend guidance and does not supply a complete frontend testing method.

For customer data, prioritize tenant isolation, role permissions, preservation of untouched fields during mutations, and both sides of feature flags. Follow the checkout's requirements for these cases.

## Keep tests readable and reproducible

Co-locate tests with their subject, usually as `*.test.ts`. Use `describe` for the subject and context, and `it` for a specific outcome. Prefer precise assertions with useful failure messages. Follow the current package's assertion library and type-narrowing conventions.

Use `beforeEach` for shared setup that must reset between cases. Reserve suite-wide setup for resources that can safely be shared. Clean up in the matching lifecycle hook. Do not let one case depend on another's records or side effects.

Use explicit cases for meaningful edge values. Do not depend on chance-generated values for the expected behavior. Pin time for date-sensitive tests and avoid assumptions about database result order. Await all work before cleanup.

Run test commands from the repository root. Confirm their current definitions before use. Common entry points are `just unit-test <path>` and scoped Turbo tasks. Follow repository logging and workspace-selection rules. Never leave `it.only`, `describe.only`, debug logs, or unintended skips in the change.

If a promise-count performance check fails, investigate the cause before updating the threshold. A changed count alone does not establish either a performance regression or an improvement.
