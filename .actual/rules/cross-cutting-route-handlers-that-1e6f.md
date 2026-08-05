# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Route Handlers That

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

### Rules

- **R-CONC-001** MUST: Route handlers that execute multiple database operations within a single request MUST use database transaction boundaries to ensure atomicity and isolation.

### Verify

```bash
# Discover the project's test suite location and identify integration tests for API route handlers
find . -type f -name "*.test.*" -o -name "*.spec.*" | grep -i route | head -20

# Execute tests that verify concurrent request handling and transaction isolation
# (Adjust test runner command based on project's build tool)
npm test -- --testPathPattern="route|handler|concurrent" 2>&1 | head -50

# Locate the project's static analysis configuration and run linting rules
find . -type f \( -name ".eslintrc*" -o -name "eslint.config.*" -o -name "tsconfig.json" \) | head -5

# Identify the database query logging configuration
grep -r "transaction\|query.*log\|debug.*db" . --include="*.ts" --include="*.js" --include="*.env*" 2>/dev/null | head -20

# Search for route handlers and verify transaction usage
grep -r "router\.\(post\|patch\|delete\|get\)\|@\(Post\|Patch\|Delete\|Get\)" . --include="*.ts" --include="*.js" -A 10 | grep -E "transaction|beginTransaction|prisma\.\$transaction" | head -30
```

**Accept when:**
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers performing database operations MUST be wrapped in explicit transaction boundaries before code review approval.
</enforcement>