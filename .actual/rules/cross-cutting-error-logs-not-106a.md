# Standardize Error Logging with Console Error in API Routes: Error Logs Not

These rules are ALWAYS ACTIVE for all API route handlers in Next.js server-side routes, error handling blocks in database query operations, authentication and authorization failure paths, external service integration error handling, and input validation failure logging.

### Rules

- **R-ERRLOG-001** MUST NOT: Error logs MUST NOT include plaintext passwords, raw authentication tokens, or unredacted personally identifiable information.
- **R-ERRLOG-002** MUST: Wrap all database query operations, external service calls, and authentication checks in try-catch blocks that log errors before returning appropriate HTTP error responses.
- **R-ERRLOG-003** MUST: Structure error context strings to identify the operation type and resource without including user input or sensitive identifiers that could leak information.
- **R-ERRLOG-004** SHOULD: Review existing API route error handlers to ensure consistent error logging patterns and identify any gaps in error visibility coverage.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration to verify error logging patterns in API route handlers
find . -name ".eslintrc*" -o -name "eslint.config.*" | head -1

# Locate the project's test suite and run integration tests that verify error logging behavior in failure scenarios
find . -type f \( -name "*.test.ts" -o -name "*.test.js" -o -name "*.spec.ts" -o -name "*.spec.js" \) | grep -i "api\|route" | head -5

# Inspect the project's API route files to confirm console error calls exist in catch blocks
find . -path "*/pages/api/*" -o -path "*/app/api/*" | xargs grep -l "console\.error" 2>/dev/null | head -10

# Verify error logging patterns in catch blocks
find . -path "*/pages/api/*" -o -path "*/app/api/*" | xargs grep -A 2 "catch" 2>/dev/null | grep -c "console\.error"
```

**Accept when:**
- All API route handlers contain console error logging in catch blocks for database operations, authentication failures, and external service errors.
- Error log messages include descriptive context strings and error objects without exposing sensitive data.
- Code review checklist includes verification of error logging patterns and sensitive data redaction.
- No error logs are found to contain plaintext passwords, raw authentication tokens, or unredacted personally identifiable information.

<enforcement>
Claude Code MUST NOT skip or defer verification. All API route handlers must be inspected for compliance with R-ERRLOG-001 through R-ERRLOG-004 before code is committed.
</enforcement>