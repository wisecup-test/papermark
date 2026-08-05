# Validate Input Before Logging Error Context: Route Handlers Log

These rules are ALWAYS ACTIVE for all API route handlers that perform input validation and error logging, including handlers that accept POST, GET, PATCH, or DELETE requests with path parameters, query parameters, or request bodies.

### Rules

- **R-VALIDATE-LOG-001** MUST: Validate all external input using schema parsers before including request-derived data in error logging statements.
- **R-VALIDATE-LOG-002** MUST: Log only validated identifiers and parameters in error context; do not echo raw or partially validated input values.
- **R-VALIDATE-LOG-003** MAY: Route handlers MAY log sanitized metadata about validation failures for security monitoring purposes.
- **R-VALIDATE-LOG-004** MUST: Extract validated identifiers into typed variables immediately after validation to create clear boundaries between validated and unvalidated data.
- **R-VALIDATE-LOG-005** MUST: For database errors, log the validated identifier used in the query rather than re-extracting it from the request.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linter
# to detect console logging statements that reference request parameters before validation
linter_config=$(find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1)
if [ -n "$linter_config" ]; then
  npm run lint -- --format json > lint-report.json 2>&1
fi

# Locate the project's test suite directory and run integration tests
# that verify validation failures do not log raw input values
test_dir=$(find . -type d -name '__tests__' -o -name 'tests' -o -name 'test' | head -1)
if [ -n "$test_dir" ]; then
  npm test -- "$test_dir" --testPathPattern='validation|error.*log' 2>&1
fi

# Identify the repository's code search tooling and scan for error logging patterns
# that include request-derived data, then manually verify each occurs after validation
grep -r 'console\.error\|console\.log' --include='*.ts' --include='*.js' \
  | grep -E 'req\.(params|query|body)' \
  | head -20
```

**Accept when:**
- All error logging statements in API route handlers reference only validated identifiers and parameters
- Static analysis reports zero violations of logging-before-validation patterns
- Integration tests confirm validation failures produce logs without echoing invalid input
- Code review verification confirms error logs only include validated data
- No console logging statements reference request-derived data before schema validation checkpoints

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for route handlers in scope. Violations must be caught during code review and static analysis before merge.
</enforcement>