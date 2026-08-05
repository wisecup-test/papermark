# Validate Input Before Logging Error Context: Error Logging Statements

These rules are ALWAYS ACTIVE for all API route handlers that perform input validation and error logging, including handlers that accept POST, GET, PATCH, or DELETE requests with path parameters, query parameters, or request bodies.

### Rules

- **R-ELG-001** MUST: Error logging statements MUST only include validated identifiers and parameters that have passed schema parsing.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linter to detect console logging statements that reference request parameters before validation
linter_config=$(find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1)
if [ -n "$linter_config" ]; then
  npm run lint -- --format json > lint-report.json 2>&1
fi

# Locate the project's test suite directory and run integration tests that verify validation failures do not log raw input values
test_dir=$(find . -type d -name '__tests__' -o -name 'test' -o -name 'tests' | head -1)
if [ -n "$test_dir" ]; then
  npm test -- "$test_dir" --testPathPattern='validation|error.*log' 2>&1
fi

# Identify the repository's code search tooling and scan for error logging patterns that include request-derived data
grep -r 'console\.error\|console\.log' --include='*.ts' --include='*.js' . | grep -E '(req\.|params\.|query\.|body\.)' | head -20
```

**Accept when:**
- All error logging statements in API route handlers reference only validated identifiers and parameters
- Static analysis reports zero violations of logging-before-validation patterns
- Integration tests confirm validation failures produce logs without echoing invalid input
- Code review verification confirms each error logging statement occurs after a validation checkpoint (schema parsing, safeParse, or equivalent)

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands MUST be executed before accepting changes. Build failure or test failure indicates violation of R-ELG-001.
</enforcement>