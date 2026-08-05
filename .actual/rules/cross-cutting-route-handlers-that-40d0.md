# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Route Handlers That

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

### Rules

- **R-HTTP-001** MUST: All API route handlers that perform database write operations (create, update, delete) MUST use HTTP methods that signal write intent (POST, PATCH, PUT, DELETE) and MUST NOT use GET for state-modifying operations.
- **R-HTTP-002** MUST: All route handlers that perform authorization checks MUST ensure that authorization logic executes within the same transaction as subsequent database queries to prevent time-of-check-time-of-use races.
- **R-HTTP-003** MUST: Multi-step database operations within route handlers MUST execute within explicit transaction boundaries with proper rollback on error and commit on success.
- **R-HTTP-004** SHOULD: Establish consistent lock ordering conventions across all route handlers to prevent deadlocks when multiple transactions acquire locks in different orders.
- **R-HTTP-005** SHOULD: Set transaction timeout limits and move expensive operations (external API calls, file processing) outside transaction boundaries to prevent long-running transactions from holding database locks.

### Verify

```bash
# Discover the project's test suite location and identify integration tests for API route handlers
find . -type f -name "*.test.*" -o -name "*.spec.*" | grep -E "(route|handler|api)" | head -20

# Execute integration tests that verify concurrent request handling and transaction isolation
npm test -- --testPathPattern="(route|handler|api)" --testNamePattern="concurrent|transaction|isolation"

# Locate the project's static analysis configuration and run linting rules
find . -maxdepth 2 -type f \( -name ".eslintrc*" -o -name "eslint.config.*" -o -name "biome.json" \)

# Run static analysis to detect database operations outside transaction boundaries
npm run lint -- --format json 2>/dev/null | grep -i "transaction\|database\|query"

# Identify the database query logging configuration
grep -r "query.*log\|transaction.*log" . --include="*.ts" --include="*.js" --include="*.env*" 2>/dev/null | head -10

# Verify HTTP method usage in route handlers
grep -r "router\.(get|post|patch|put|delete)" . --include="*.ts" --include="*.js" | grep -v node_modules | head -20

# Check for GET handlers that modify state
grep -r "router\.get" . --include="*.ts" --include="*.js" | xargs grep -l "create\|update\|delete\|insert" 2>/dev/null
```

**Accept when:**
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error
- Authorization checks execute within the same transaction as subsequent database queries
- Request-scoped transaction management automatically rolls back on unhandled exceptions and commits on successful response generation

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers MUST be audited for HTTP method semantics, transaction boundaries, and authorization check placement before code review approval.
</enforcement>