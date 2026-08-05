# Enforce Zod Schema Validation for All API Route Input Parameters: Validation Schemas Defined

These rules are ALWAYS ACTIVE for all HTTP route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

### Rules

- **R-VAL-001** SHOULD: Validation schemas SHOULD be defined as reusable schema objects and imported into route handlers rather than defined inline.

### Verify

```bash
# Discover the project's test runner configuration and execute the test suite covering API route validation logic
# (Exact command depends on project's build tool and test runner — inspect package.json or build manifest)

# Discover the project's static analysis or linting configuration and execute checks that enforce validation patterns
# (Exact command depends on project's linter configuration)

# Discover the project's integration test suite and verify tests exist that submit invalid input to API routes and assert validation failures
# (Exact command depends on project's test framework)
```

**Accept when:**
- All API route handlers that accept external input include schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes
- Validation schemas are defined in dedicated schema modules organized by domain or feature area
- Safe parsing methods are used that return success or failure results before accessing validated data
- Appropriate HTTP status codes are returned for validation failures with descriptive error messages
- Array parameters include maximum length constraints to prevent resource exhaustion

<enforcement>
Claude Code MUST NOT skip or defer verification. All API route handlers accepting external input MUST include schema validation before processing, and validation MUST complete before any database queries execute.
</enforcement>