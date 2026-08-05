# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Route Handlers That

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations, including endpoints that perform authentication, authorization, and business logic operations within request processing pipelines.

### Rules

- **R-CONC-001** SHOULD: Route handlers that access shared resources (database records, cache entries, external services) SHOULD implement optimistic or pessimistic locking strategies appropriate to the operation's concurrency requirements.

### Verify

```bash
# Discover the project's test suite location and identify integration tests for API route handlers
find . -type f -name "*.test.*" -o -name "*.spec.*" | grep -i route | head -20

# Execute tests that verify concurrent request handling and transaction isolation
npm test -- --testPathPattern="(route|handler|concurrent|transaction)" 2>&1 | head -50

# Locate the project's static analysis configuration and run linting rules
if [ -f ".eslintrc" ] || [ -f ".eslintrc.json" ] || [ -f "eslint.config.js" ]; then
  npx eslint --format=json src/**/*.ts src/**/*.js 2>&1 | grep -i "transaction\|lock\|concurrency" || echo "No concurrency violations found"
fi

# Identify the database query logging configuration and enable transaction boundary logging
grep -r "transaction\|Transaction" . --include="*.ts" --include="*.js" --include="*.env*" 2>/dev/null | grep -v node_modules | head -20

# Review route handlers for multi-step database operations
grep -r "findUnique\|create\|update\|delete" . --include="*.ts" --include="*.js" -B 2 -A 2 2>/dev/null | grep -v node_modules | head -50
```

**Accept when:**
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error
- Authorization checks execute within the same transaction as subsequent database queries to prevent time-of-check-time-of-use races
- Request-scoped transaction management automatically rolls back on unhandled exceptions and commits on successful response generation

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers performing database operations MUST be reviewed for explicit concurrency control implementation. Integration tests MUST be executed to verify transaction isolation guarantees before accepting changes.
</enforcement>