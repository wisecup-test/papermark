# Standardize Error Logging with Console Error in API Routes: Route Handlers Log

These rules are ALWAYS ACTIVE for all API route handlers in Next.js server-side routes that handle authentication, database operations, business logic failures, and external service integrations.

### Rules

- **R-ROUTE-LOG-001** MUST: All API route handlers MUST log caught exceptions and error conditions using console error output before returning error responses to clients.
- **R-ROUTE-LOG-002** MUST: Wrap all database query operations, external service calls, and authentication checks in try-catch blocks that log errors before returning appropriate HTTP error responses.
- **R-ROUTE-LOG-003** MUST: Structure error context strings to identify the operation type and resource without including user input or sensitive identifiers that could leak information.
- **R-ROUTE-LOG-004** SHOULD: Review existing API route error handlers to ensure consistent error logging patterns and identify any gaps in error visibility coverage.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration to verify error logging patterns in API route handlers
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1

# Locate the project's test suite and run integration tests that verify error logging behavior in failure scenarios
find . -type f -name '*.test.*' -o -name '*.spec.*' | grep -E '(api|route)' | head -5

# Inspect the project's API route files to confirm console error calls exist in catch blocks
grep -r "console\.error" --include="*.ts" --include="*.js" | grep -E "(api|route)" | head -10

# Verify error logging in catch blocks for database operations
grep -r "catch.*{" --include="*.ts" --include="*.js" -A 3 | grep -E "(prisma|database|query)" | head -10
```

**Accept when:**
- All API route handlers contain console error logging in catch blocks for database operations, authentication failures, and external service errors
- Error log messages include descriptive context strings and error objects without exposing sensitive data
- Code review checklist includes verification of error logging patterns and sensitive data redaction
- Static analysis rules detect missing error logging in catch blocks
- Security review validates that error logs do not expose sensitive data

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests missing error logging in API route error handlers are blocked until logging is added. Security team reviews any error logs found to contain sensitive data and requires immediate remediation.
</enforcement>