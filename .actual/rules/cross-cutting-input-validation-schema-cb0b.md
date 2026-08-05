# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Input Validation Schema

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

### Rules

- **R-CONCUR-001** MUST: Input validation using schema parsers MUST complete successfully before any database operation begins, and validation failures MUST prevent database access.

### Verify

```bash
# Discover the project's test suite location and identify integration tests for API route handlers
find . -type f -name "*.test.*" -o -name "*.spec.*" | grep -E "(route|handler|api)" | head -20

# Execute tests that verify concurrent request handling and transaction isolation
npm test -- --testPathPattern="(route|handler|api|concurrent)" 2>&1 | tail -50

# Locate the project's static analysis configuration and run linting rules
if [ -f ".eslintrc" ] || [ -f ".eslintrc.json" ] || [ -f "eslint.config.js" ]; then
  npx eslint . --format=json 2>&1 | grep -E "(transaction|validation|database)" || echo "No linting violations found"
fi

# Identify the database query logging configuration and enable transaction boundary logging
grep -r "transaction\|Transaction" . --include="*.ts" --include="*.js" | grep -E "(begin|start|commit|rollback)" | head -20

# Verify no GET handlers modify state
grep -r "router\.get\|app\.get" . --include="*.ts" --include="*.js" -A 10 | grep -E "(create|update|delete|save|remove)" | head -20
```

**Accept when:**
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error
- Input validation schema parsers (safeParse, parse) are invoked before any database query in all route handlers
- Authorization checks execute within the same transaction as subsequent database queries
- Request-scoped transaction management automatically rolls back on unhandled exceptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers MUST be inspected for validation-before-execution ordering and transaction boundaries before code is committed.
</enforcement>