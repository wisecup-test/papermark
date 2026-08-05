# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Get Route Handlers

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations, particularly GET handlers that perform read-only database queries.

### Rules

- **R-GET-001** SHOULD: GET route handlers that perform read-only database queries SHOULD use appropriate isolation levels to prevent dirty reads and ensure consistent snapshots.

### Verify

```bash
# Discover the project's test suite location and identify integration tests for API route handlers
find . -type f -name "*.test.*" -o -name "*.spec.*" | grep -i route | head -20

# Execute tests that verify concurrent request handling and transaction isolation
npm test -- --testPathPattern="(route|handler|api)" --testNamePattern="(concurrent|isolation|transaction)"

# Locate the project's static analysis configuration and run linting rules
find . -type f \( -name ".eslintrc*" -o -name "eslint.config.*" -o -name "tsconfig.json" \) | head -5

# Run linting to detect database operations outside transaction boundaries
npm run lint

# Identify the database query logging configuration
grep -r "query.*log\|log.*query\|transaction.*log" . --include="*.ts" --include="*.js" --include="*.env*" | head -10

# Review database transaction logs for handlers that perform multiple queries
grep -r "findUnique\|findMany\|create\|update\|delete" . --include="*.ts" --include="*.js" | grep -v node_modules | head -20
```

**Accept when:**
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error
- GET route handlers explicitly specify isolation levels (READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, or SERIALIZABLE) appropriate to their consistency requirements

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers must be inspected for HTTP method semantics, transaction boundaries, and isolation level configuration before approval.
</enforcement>