# Enforce Zod Schema Validation for All API Route Input Parameters: Validation Use Safe

These rules are ALWAYS ACTIVE for all API route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

### Rules

- **R-VAL-001** MUST: Validation MUST use safe parsing methods that return success or failure results rather than throwing exceptions, allowing explicit error handling.
- **R-VAL-002** MUST: All HTTP route handlers that accept POST, PATCH, PUT, or DELETE requests with request bodies MUST include schema validation before processing.
- **R-VAL-003** MUST: All route handlers that read query parameters from URLs MUST include schema validation before processing.
- **R-VAL-004** MUST: All route handlers that extract path parameters from dynamic route segments MUST include schema validation before processing.
- **R-VAL-005** MUST: All route handlers that read cookie values for authentication or session management MUST include schema validation before processing.
- **R-VAL-006** MUST: Validation of identifiers used in database queries including primary keys and foreign keys MUST occur before any database query executes.
- **R-VAL-007** MUST: For endpoints that accept arrays, MUST always specify maximum length constraints to prevent resource exhaustion from oversized payloads.
- **R-VAL-008** MUST: Validation schemas MUST be defined in dedicated schema modules organized by domain or feature area to promote reuse across related endpoints.
- **R-VAL-009** MUST: Return appropriate HTTP status codes for validation failures with error messages that describe what validation failed without exposing internal implementation details.

### Verify

```bash
# Discover the project's test runner configuration and execute the test suite covering API route validation logic
# (Exact command depends on project's build tool and test runner — derive from package.json or build manifest)

# Discover the project's static analysis or linting configuration and execute checks that enforce validation patterns
# (Exact command depends on project's linter configuration — derive from project root)

# Discover the project's integration test suite and verify tests exist that submit invalid input to API routes and assert validation failures
# (Exact command depends on project's test framework — derive from test configuration)

# Verify no database queries execute before validation completes by inspecting route handler source code
grep -r "safeParse\|parse" src/ | grep -v "node_modules"
```

**Accept when:**
- All API route handlers that accept external input include schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes
- All array-accepting endpoints specify maximum length constraints in their validation schemas
- Validation schemas are organized in dedicated modules by domain or feature area
- HTTP error responses for validation failures do not expose internal implementation details

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API route handlers MUST include schema validation using safe parsing methods before any business logic or database query execution. Pull requests without input validation are blocked until validation is added.
</enforcement>