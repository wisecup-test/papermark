# Enforce Zod Schema Validation for All API Route Input Parameters: Array Inputs Specify

These rules are ALWAYS ACTIVE for all HTTP route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

### Rules

- **R-ZVAL-001** MUST: All HTTP route handlers that accept POST, PATCH, PUT, or DELETE requests with request bodies SHALL include schema validation before processing.
- **R-ZVAL-002** MUST: All route handlers that read query parameters from URLs SHALL include schema validation before processing.
- **R-ZVAL-003** MUST: All route handlers that extract path parameters from dynamic route segments SHALL include schema validation before processing.
- **R-ZVAL-004** MUST: All route handlers that read cookie values for authentication or session management SHALL include schema validation before processing.
- **R-ZVAL-005** MUST: Validation of identifiers used in database queries including primary keys and foreign keys SHALL occur before any database query executes.
- **R-ZVAL-006** SHOULD: Array inputs SHOULD specify minimum and maximum length constraints to prevent resource exhaustion attacks.
- **R-ZVAL-007** MUST: Validation schemas SHALL use safe parsing methods that return success or failure results without throwing exceptions.
- **R-ZVAL-008** MUST: Validation failures SHALL be checked before accessing validated data to ensure type safety.
- **R-ZVAL-009** MUST: Validation failures SHALL return appropriate HTTP status codes with error messages that describe what validation failed without exposing internal implementation details.

### Verify

```bash
# Discover and run the project's test suite covering API route validation logic
npm test -- --testPathPattern="(route|api|validation)" 2>&1 | head -50

# Discover and run static analysis or linting configuration that enforces validation patterns
npm run lint 2>&1 | grep -i "validation\|schema" | head -20

# Verify integration tests exist that submit invalid input to API routes
grep -r "safeParse\|validation.*fail\|invalid.*input" --include="*.test.ts" --include="*.spec.ts" . 2>/dev/null | head -20

# Verify no database queries execute before validation
grep -B5 "db\.\|query\|execute" --include="*.ts" --include="*.js" . 2>/dev/null | grep -A5 "safeParse" | head -30
```

**Accept when:**
- All API route handlers that accept external input include schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes
- Array inputs in validation schemas specify both minimum and maximum length constraints
- Validation uses safe parsing methods that return results rather than throwing exceptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory; rules marked SHOULD are strongly preferred. Verification commands MUST be executed before accepting changes that modify API route handlers or validation schemas.
</enforcement>