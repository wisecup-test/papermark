# Standardize Error Logging with Console Error in API Routes: Error Context Strings

These rules are ALWAYS ACTIVE for all API route handlers in Next.js server-side routes, error handling blocks in database query operations, authentication and authorization failure paths, and external service integration error handling.

### Rules

- **R-ERR-001** SHOULD: Error context strings SHOULD identify the specific resource or operation type to enable log aggregation and filtering.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration to verify error logging patterns in API route handlers
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1

# Locate the project's test suite and run integration tests that verify error logging behavior in failure scenarios
find . -type f \( -name 'jest.config.*' -o -name 'vitest.config.*' -o -name 'package.json' \) | grep -E '(jest|vitest|test)' | head -1

# Inspect the project's API route files to confirm console error calls exist in catch blocks for database operations and authentication failures
find . -path '*/pages/api/*' -o -path '*/app/api/*' | xargs grep -l 'console\.error' 2>/dev/null | head -10
```

**Accept when:**
- All API route handlers contain console error logging in catch blocks for database operations, authentication failures, and external service errors
- Error log messages include descriptive context strings and error objects without exposing sensitive data
- Code review checklist includes verification of error logging patterns and sensitive data redaction

<enforcement>
Claude Code MUST NOT skip or defer verification. All API route error handlers must be audited for compliance with R-ERR-001 before accepting changes.
</enforcement>