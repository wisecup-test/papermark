# Enforce Zod Schema Validation for All API Route Input Parameters: Cuid Identifiers Path

These rules are ALWAYS ACTIVE for all API route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

### Rules

- **R-CUID-001** MUST: CUID identifiers in path parameters and request bodies MUST be validated against the CUID format specification before any business logic or database query execution.
- **R-CUID-002** MUST: All HTTP route handlers that accept POST, PATCH, PUT, or DELETE requests with request bodies MUST include schema validation using safe parsing methods that return success/failure results without throwing exceptions.
- **R-CUID-003** MUST: All route handlers that read query parameters from URLs MUST validate parameters against defined schemas before processing.
- **R-CUID-004** MUST: All route handlers that extract path parameters from dynamic route segments MUST validate identifiers against the CUID format specification.
- **R-CUID-005** MUST: All route handlers that read cookie values for authentication or session management MUST validate cookie values against defined schemas.
- **R-CUID-006** MUST: Validation of identifiers used in database queries including primary keys and foreign keys MUST occur before the query executes.
- **R-CUID-007** MUST: Validation schemas MUST be defined in dedicated schema modules organized by domain or feature area to promote reuse across related endpoints.
- **R-CUID-008** MUST: Safe parsing methods MUST be used with explicit checks of the success property before accessing validated data to ensure type safety.
- **R-CUID-009** MUST: HTTP status codes for validation failures MUST be appropriate with error messages that describe what validation failed without exposing internal implementation details.
- **R-CUID-010** MUST: For endpoints that accept arrays, maximum length constraints MUST always be specified to prevent resource exhaustion from oversized payloads.

### Verify

```bash
# Discover and run the project's test suite covering API route validation logic
test_runner_config=$(find . -name "*.config.*" -o -name "jest.config.*" -o -name "vitest.config.*" | head -1)
if [ -n "$test_runner_config" ]; then
  npm test -- --testPathPattern="(route|api|validation)" 2>&1 | grep -E "(PASS|FAIL|validation)"
fi

# Discover and run static analysis or linting configuration
lint_config=$(find . -name ".eslintrc*" -o -name "eslint.config.*" | head -1)
if [ -n "$lint_config" ]; then
  npm run lint -- --format=json 2>&1 | grep -E "(validation|schema|cuid)" || echo "No validation pattern violations found"
fi

# Verify integration tests exist for invalid input submission
grep -r "safeParse\|validation\|schema" --include="*.test.*" --include="*.spec.*" . 2>/dev/null | wc -l

# Check for database queries before validation patterns
grep -r "db\.query\|prisma\.\|orm\." --include="*.ts" --include="*.js" . 2>/dev/null | grep -v "safeParse" | head -5
```

**Accept when:**
- All API route handlers that accept external input include schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes
- CUID identifiers in path parameters are validated against the CUID format specification
- Safe parsing methods return success/failure results that are explicitly checked before data access
- Validation schemas are organized in dedicated modules by domain or feature area
- Array endpoints specify maximum length constraints in their validation schemas

<enforcement>
Claude Code MUST NOT skip or defer verification. All API route handlers accepting external input MUST include Zod schema validation with CUID format checks before any database queries or business logic execution. Validation MUST use safe parsing methods with explicit success checks. Pull requests without proper validation are blocked until remediated.
</enforcement>