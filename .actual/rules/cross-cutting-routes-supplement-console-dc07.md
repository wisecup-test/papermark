# Standardize Error Logging with Console Error in API Routes: Routes Supplement Console

These rules are ALWAYS ACTIVE for all API route handlers in Next.js server-side routes, error handling blocks in database query operations, authentication and authorization failure paths, and external service integration error handling.

### Rules

- **R-CONSOLE-001** MUST: Wrap all database query operations, external service calls, and authentication checks in try-catch blocks that log errors using console.error before returning appropriate HTTP error responses.
- **R-CONSOLE-002** MUST: Structure error context strings to identify the operation type and resource without including user input or sensitive identifiers that could leak information.
- **R-CONSOLE-003** MAY: API routes MAY supplement console error logging with structured logging libraries for enhanced metadata capture.
- **R-CONSOLE-004** MUST: Ensure error log messages include descriptive context strings and error objects without exposing sensitive data.
- **R-CONSOLE-005** SHOULD: Review existing API route error handlers to ensure consistent error logging patterns and identify any gaps in error visibility coverage.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration to verify error logging patterns in API route handlers
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1

# Locate the project's test suite and run integration tests that verify error logging behavior in failure scenarios
find . -path '*/test*' -o -path '*/spec*' -o -path '*/__tests__' | grep -E '\.(test|spec)\.(ts|js)$' | head -5

# Inspect the project's API route files to confirm console error calls exist in catch blocks
grep -r 'console\.error' --include='*.ts' --include='*.js' | grep -E '(pages/api|app/api|routes)' | head -10

# Verify catch blocks exist in API route handlers
grep -r 'catch\s*(' --include='*.ts' --include='*.js' | grep -E '(pages/api|app/api|routes)' | head -10
```

**Accept when:**
- All API route handlers contain console.error logging in catch blocks for database operations, authentication failures, and external service errors
- Error log messages include descriptive context strings and error objects without exposing sensitive data
- Code review checklist includes verification of error logging patterns and sensitive data redaction
- Static analysis rules detect missing error logging in catch blocks
- Security review validates that error logs do not expose sensitive data

<enforcement>
Claude Code MUST NOT skip or defer verification. All API route handlers MUST include console.error logging in catch blocks. Pull requests missing error logging in API route error handlers are blocked until logging is added. Security team reviews any error logs found to contain sensitive data and requires immediate remediation.
</enforcement>