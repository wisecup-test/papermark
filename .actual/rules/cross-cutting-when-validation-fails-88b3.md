# Validate Input Before Logging Error Context: When Validation Fails

These rules are ALWAYS ACTIVE for all API route handlers that perform input validation and error logging, including handlers that accept POST, GET, PATCH, or DELETE requests with path parameters, query parameters, or request bodies.

### Rules

- **R-VAL-001** SHOULD: When validation fails, error logs SHOULD record the validation failure type and field path without echoing the invalid input value.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linter to detect console logging statements that reference request parameters before validation
# (Exact command depends on project's linter configuration — see project repository)

# Locate the project's test suite directory and run integration tests that verify validation failures do not log raw input values
# (Exact command depends on project's test runner — see project repository)

# Identify the repository's code search tooling and scan for error logging patterns that include request-derived data, then manually verify each occurs after validation
# (Exact command depends on project's code search tool — see project repository)
```

**Accept when:**
- All error logging statements in API route handlers reference only validated identifiers and parameters
- Static analysis reports zero violations of logging-before-validation patterns
- Integration tests confirm validation failures produce logs without echoing invalid input

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification steps MUST be executed before accepting changes that modify error logging or validation logic in API route handlers.
</enforcement>