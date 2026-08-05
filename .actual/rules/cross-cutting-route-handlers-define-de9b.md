# Enforce Zod Schema Validation for All API Route Input Parameters: Route Handlers Define

These rules are ALWAYS ACTIVE for all API route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

### Rules

- **R-ZOD-001** MUST: All HTTP route handlers that accept POST, PATCH, PUT, or DELETE requests with request bodies SHALL define and apply Zod schema validation before processing input.
- **R-ZOD-002** MUST: All route handlers that read query parameters from URLs SHALL define and apply Zod schema validation before processing input.
- **R-ZOD-003** MUST: All route handlers that extract path parameters from dynamic route segments SHALL define and apply Zod schema validation before processing input.
- **R-ZOD-004** MUST: All route handlers that read cookie values for authentication or session management SHALL define and apply Zod schema validation before processing input.
- **R-ZOD-005** MUST: Validation of identifiers used in database queries including primary keys and foreign keys SHALL be performed via Zod schema validation.
- **R-ZOD-006** MUST: Validation schemas SHALL use safe parsing methods (safeParse) that return success/failure results without throwing exceptions.
- **R-ZOD-007** MUST: Database queries and business logic processing SHALL NOT execute before validation completes and returns success.
- **R-ZOD-008** MUST: Validation failures SHALL return appropriate HTTP status codes with error messages that describe what validation failed without exposing internal implementation details.
- **R-ZOD-009** MUST: For endpoints that accept arrays, validation schemas SHALL always specify maximum length constraints to prevent resource exhaustion from oversized payloads.
- **R-ZOD-010** MAY: Route handlers MAY define separate schemas for authenticated versus unauthenticated request contexts when validation requirements differ.
- **R-ZOD-011** MUST: Validation schemas SHALL be defined in dedicated schema modules organized by domain or feature area to promote reuse across related endpoints.

### Verify

```bash
# Discover the project's test runner configuration and execute the test suite covering API route validation logic
# (Exact command depends on project's build tool and test runner — derive from package.json or build manifest)

# Discover the project's static analysis or linting configuration and execute checks that enforce validation patterns
# (Exact command depends on project's linter configuration — derive from project root)

# Discover the project's integration test suite and verify tests exist that submit invalid input to API routes and assert validation failures
# (Exact command depends on project's test framework — derive from test configuration)

# Verify no database queries execute before validation by inspecting route handler source code
grep -r "safeParse" src/routes/ && echo "Validation schemas found"

# Verify validation occurs before database operations
grep -B5 "db\." src/routes/ | grep -E "(safeParse|success)" && echo "Validation precedes DB queries"
```

**Accept when:**
- All API route handlers that accept external input include Zod schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes
- All array validation schemas include maximum length constraints
- Validation schemas are organized in dedicated modules by domain or feature area
- Safe parsing methods are used consistently across all validated endpoints
- Validation failures return appropriate HTTP status codes with descriptive error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API route handlers MUST include Zod schema validation before processing external input. Pull requests without validation are blocked until validation is added.
</enforcement>