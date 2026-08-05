# Validate Input Before Logging Error Context: Error Logs Not

These rules are ALWAYS ACTIVE for all API route handlers that perform input validation and error logging, including POST, GET, PATCH, and DELETE request handlers that accept path parameters, query parameters, or request bodies.

### Rules

- **R-VALIDATE-LOG-001** MUST_NOT: Error logs MUST NOT include raw request body content, query parameters, or path parameters before validation has confirmed their structure and type.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linter to detect console logging statements that reference request parameters before validation
lint_config=$(find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1)
if [ -n "$lint_config" ]; then npm run lint -- --format json > lint-report.json 2>&1; fi

# Locate the project's test suite directory and run integration tests that verify validation failures do not log raw input values
test_dir=$(find . -type d -name '__tests__' -o -name 'test' -o -name 'tests' | head -1)
if [ -n "$test_dir" ]; then npm test -- "$test_dir" 2>&1; fi

# Identify the repository's code search tooling and scan for error logging patterns that include request-derived data
grep -r "console\.error\|console\.log" --include="*.ts" --include="*.js" . | grep -E "req\.|query\.|params\.|body\." | head -20
```

**Accept when:**
- All error logging statements in API route handlers reference only validated identifiers and parameters
- Static analysis reports zero violations of logging-before-validation patterns
- Integration tests confirm validation failures produce logs without echoing invalid input
- Code review verification confirms error logs only include validated data structures

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands MUST be executed and their results reviewed before accepting changes to error logging in route handlers.
</enforcement>