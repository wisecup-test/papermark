# Enforce Zod Schema Validation for All API Route Input Parameters: Route Handlers Reject

These rules are ALWAYS ACTIVE for all API route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

### Rules

- **R-ROUTE-VAL-001** MUST: Route handlers MUST reject requests with validation failures before executing database queries, external API calls, or business logic operations.
- **R-ROUTE-VAL-002** MUST: All HTTP route handlers that accept POST, PATCH, PUT, or DELETE requests with request bodies MUST include schema validation.
- **R-ROUTE-VAL-003** MUST: All route handlers that read query parameters from URLs MUST include schema validation.
- **R-ROUTE-VAL-004** MUST: All route handlers that extract path parameters from dynamic route segments MUST include schema validation.
- **R-ROUTE-VAL-005** MUST: All route handlers that read cookie values for authentication or session management MUST include schema validation.
- **R-ROUTE-VAL-006** MUST: Validation of identifiers used in database queries including primary keys and foreign keys MUST occur before query execution.
- **R-ROUTE-VAL-007** MUST: Route handlers MUST use safe parsing methods that return success or failure results and check the success property before accessing validated data.
- **R-ROUTE-VAL-008** MUST: Route handlers MUST return appropriate HTTP status codes for validation failures with error messages that describe what validation failed without exposing internal implementation details.
- **R-ROUTE-VAL-009** MUST: For endpoints that accept arrays, route handlers MUST specify maximum length constraints to prevent resource exhaustion from oversized payloads.
- **R-ROUTE-VAL-010** SHOULD: Validation schemas SHOULD be defined in dedicated schema modules organized by domain or feature area to promote reuse across related endpoints.

### Verify

```bash
# Discover the project's test runner configuration and execute the test suite covering API route validation logic
# (Exact command depends on project's build tool and test runner — derive from package.json or build manifest)

# Discover the project's static analysis or linting configuration and execute checks that enforce validation patterns
# (Exact command depends on project's linter configuration — derive from project root)

# Discover the project's integration test suite and verify tests exist that submit invalid input to API routes and assert validation failures
# (Exact command depends on project's test framework — derive from test configuration)
```

**Accept when:**
- All API route handlers that accept external input include schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes
- Validation schemas are defined in dedicated modules and reused across related endpoints
- HTTP error responses for validation failures include descriptive messages without exposing implementation details
- Array parameters include maximum length constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API route handlers MUST be reviewed to confirm validation is present before database queries execute. Integration tests MUST include negative test cases for invalid input. Static analysis MUST confirm no violations of validation ordering.
</enforcement>