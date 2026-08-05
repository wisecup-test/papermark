# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Error Handling Route

These rules are ALWAYS ACTIVE for all HTTP route handlers that process HTTP requests and perform database operations, including endpoints that perform authentication, authorization, and business logic operations within request processing pipelines that include validation, database access, and response generation stages.

### Rules

- **R-CONC-001** MUST: Error handling in route handlers MUST ensure that partial database operations are rolled back and that error responses do not leak sensitive information about database state or structure.

### Verify

```bash
# Discover the project's test suite location and identify integration tests for API route handlers
find . -type f -name "*.test.*" -o -name "*.spec.*" | grep -E "(route|handler|api)" | head -20

# Execute tests that verify concurrent request handling and transaction isolation
npm test -- --testPathPattern="(route|handler|api)" --testNamePattern="(concurrent|transaction|isolation)" 2>&1 | head -50

# Locate the project's static analysis configuration and run linting rules
find . -maxdepth 2 -type f \( -name ".eslintrc*" -o -name "eslint.config.*" -o -name "biome.json" \)

# Identify the database query logging configuration
grep -r "transaction\|rollback\|commit" . --include="*.ts" --include="*.js" --include="*.env*" 2>/dev/null | grep -E "(log|debug|trace)" | head -20

# Review route handlers for explicit transaction boundaries
grep -r "\$transaction\|beginTransaction\|transaction(" . --include="*.ts" --include="*.js" | head -30
```

**Accept when:**
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error
- Error responses do not expose database schema, query structure, or internal error details
- Partial database operations are rolled back on error conditions

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers must be inspected for transaction boundaries and error handling patterns before approval.
</enforcement>