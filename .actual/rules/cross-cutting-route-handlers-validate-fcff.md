# Validate Input Before Logging Error Context: Route Handlers Validate

These rules are ALWAYS ACTIVE for all API route handlers that perform input validation and error logging, including handlers that accept POST, GET, PATCH, or DELETE requests with path parameters, query parameters, or request bodies.

### Rules

- **R-VALIDATE-001** MUST: API route handlers MUST validate all external input using schema validation before including any input-derived values in error log statements.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linter to detect console logging statements that reference request parameters before validation

# Locate the project's test suite directory and run integration tests that verify validation failures do not log raw input values

# Identify the repository's code search tooling and scan for error logging patterns that include request-derived data, then manually verify each occurs after validation
```

**Accept when:**
- All error logging statements in API route handlers reference only validated identifiers and parameters
- Static analysis reports zero violations of logging-before-validation patterns
- Integration tests confirm validation failures produce logs without echoing invalid input

<enforcement>
Claude Code MUST NOT skip or defer verification. All error logging in route handlers must be verified to occur only after schema validation checkpoints.
</enforcement>