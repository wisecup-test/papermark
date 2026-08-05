# Standardize Error Logging with Console Error in API Routes: Error Log Calls

These rules are ALWAYS ACTIVE for all API route handlers in Next.js server-side routes, error handling blocks in database query operations, authentication and authorization failure paths, external service integration error handling, and input validation failure logging.

### Rules

- **R-ERRLOG-001** MUST: Error log calls MUST include the error object as a second parameter to preserve stack traces and error metadata.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration to verify error logging patterns in API route handlers
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1

# Locate the project's test suite and run integration tests that verify error logging behavior in failure scenarios
find . -name 'jest.config.*' -o -name 'vitest.config.*' -o -name 'package.json' | xargs grep -l '"test"' | head -1

# Inspect the project's API route files to confirm console error calls exist in catch blocks for database operations and authentication failures
find . -path '*/pages/api/*' -o -path '*/app/api/*' | head -5

# Verify console.error calls include error object as second parameter
grep -r 'console\.error' --include='*.ts' --include='*.js' | grep -E 'catch|error' | head -10
```

**Accept when:**
- All API route handlers contain console error logging in catch blocks for database operations, authentication failures, and external service errors
- Error log messages include descriptive context strings and error objects without exposing sensitive data
- Code review checklist includes verification of error logging patterns and sensitive data redaction
- Static analysis rules detect missing error logging in catch blocks
- Security review validates that error logs do not expose sensitive data

<enforcement>
Claude Code MUST NOT skip or defer verification. All API route error handlers MUST include console.error calls with error objects as the second parameter. Pull requests missing this pattern are blocked until logging is added.
</enforcement>