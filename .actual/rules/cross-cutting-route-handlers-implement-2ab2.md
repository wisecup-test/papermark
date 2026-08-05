# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Route Handlers Implement

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

### Rules

- **R-CONC-001** MAY: Route handlers MAY implement idempotency tokens for POST operations that create resources, allowing safe retry of failed requests without duplicate resource creation.

- **R-CONC-002** MUST: All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state.

- **R-CONC-003** MUST: Multi-step database operations within route handlers execute within explicit transaction boundaries with proper rollback on error.

- **R-CONC-004** MUST: Authorization checks execute within the same transaction as subsequent database queries to prevent time-of-check-time-of-use races.

- **R-CONC-005** MUST: Input validation using schema parsers occurs before database operations, establishing a validation-then-execution pattern that maintains atomicity guarantees.

- **R-CONC-006** SHOULD: Implement request-scoped transaction management that automatically rolls back on unhandled exceptions and commits on successful response generation.

### Verify

```bash
# Discover the project's test suite location and identify integration tests for API route handlers
find . -type f -name "*.test.*" -o -name "*.spec.*" | grep -E "(route|handler|api)" | head -20

# Execute tests that verify concurrent request handling and transaction isolation
npm test -- --testPathPattern="(route|handler|api|concurrent)" 2>&1 | tail -50

# Locate the project's static analysis configuration and run linting rules
find . -maxdepth 2 -type f \( -name ".eslintrc*" -o -name "eslint.config.*" -o -name "tsconfig.json" \)

# Run linting to detect database operations outside transaction boundaries
npm run lint 2>&1 | grep -E "(transaction|boundary|validation)" || echo "No transaction boundary violations detected"

# Identify the database query logging configuration
grep -r "transaction\|query.*log\|debug.*db" . --include="*.ts" --include="*.js" --include="*.env*" 2>/dev/null | head -10

# Check for route handlers that perform write operations
grep -r "router\.\(post\|patch\|delete\|put\)" . --include="*.ts" --include="*.js" 2>/dev/null | head -20

# Verify GET handlers do not modify state
grep -r "router\.get" . --include="*.ts" --include="*.js" -A 10 2>/dev/null | grep -E "(create|update|delete|save)" | head -10
```

**Accept when:**
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error
- Authorization checks are verified to execute within the same transaction as subsequent database queries
- Request-scoped transaction management is implemented with automatic rollback on unhandled exceptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-CONC-001 through R-CONC-006 must be verified before accepting route handler implementations.
</enforcement>