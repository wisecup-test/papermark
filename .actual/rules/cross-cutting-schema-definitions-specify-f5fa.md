# Enforce Zod Schema Validation for All API Route Input Parameters: Schema Definitions Specify

These rules are ALWAYS ACTIVE for all API route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

### Rules

- **R-SCHEMA-001** MUST: Schema definitions MUST specify type constraints, format requirements, and boundary conditions for all input fields including string formats, numeric ranges, array size limits, and required versus optional fields.
- **R-SCHEMA-002** MUST: All HTTP route handlers that accept POST, PATCH, PUT, or DELETE requests with request bodies MUST validate input using schema definitions before processing.
- **R-SCHEMA-003** MUST: All route handlers that read query parameters from URLs MUST validate input using schema definitions before processing.
- **R-SCHEMA-004** MUST: All route handlers that extract path parameters from dynamic route segments MUST validate input using schema definitions before processing.
- **R-SCHEMA-005** MUST: All route handlers that read cookie values for authentication or session management MUST validate input using schema definitions before processing.
- **R-SCHEMA-006** MUST: Validation of identifiers used in database queries including primary keys and foreign keys MUST occur before any database query executes.
- **R-SCHEMA-007** MUST: Use safe parsing methods that return success or failure results and check the success property before accessing validated data.
- **R-SCHEMA-008** MUST: Return appropriate HTTP status codes for validation failures with error messages that describe what validation failed without exposing internal implementation details.
- **R-SCHEMA-009** MUST: For endpoints that accept arrays, always specify maximum length constraints to prevent resource exhaustion from oversized payloads.
- **R-SCHEMA-010** SHOULD: Define validation schemas in dedicated schema modules organized by domain or feature area to promote reuse across related endpoints.

### Verify

```bash
# Discover the project's test runner configuration and execute the test suite covering API route validation logic
# (Exact command depends on project's build tool and test runner — derive from package.json or build manifest)

# Discover the project's static analysis or linting configuration and execute checks that enforce validation patterns
# (Exact command depends on project's linter configuration — derive from project root)

# Discover the project's integration test suite and verify tests exist that submit invalid input to API routes and assert validation failures
# (Exact command depends on project's test framework — derive from test configuration)

# Verify no database queries execute before validation completes by inspecting route handler source code
grep -r "safeParse\|parse" src/routes --include="*.ts" --include="*.js"
```

**Accept when:**
- All API route handlers that accept external input include schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes
- Schema definitions specify type constraints, format requirements, and boundary conditions for all input fields
- Safe parsing methods are used that return success/failure results without throwing exceptions
- Array inputs include maximum length constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. All API routes accepting external input MUST include schema validation before any business logic or database operations execute.
</enforcement>