# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Route Handlers That

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations, including endpoints that perform authentication, authorization, and business logic operations within request processing pipelines that include validation, database access, and response generation stages.

### Rules

- **R-CONC-001** MUST: Route handlers that query database entities with authorization checks MUST perform authorization validation before executing queries, and MUST NOT retrieve data that the requester is not authorized to access.
- **R-CONC-002** MUST: All route handlers that perform write operations MUST use appropriate HTTP methods (POST, PATCH, DELETE) and MUST NOT modify state in GET handlers.
- **R-CONC-003** MUST: Multi-step database operations within route handlers MUST execute within explicit transaction boundaries with proper rollback on error and commit on success.
- **R-CONC-004** MUST: Input validation using schema parsers MUST occur before database operations to establish a validation-then-execution pattern that maintains atomicity guarantees.
- **R-CONC-005** SHOULD: Route handlers that access multiple related entities SHOULD establish consistent lock ordering conventions to prevent deadlocks.
- **R-CONC-006** SHOULD: Long-running transactions in route handlers SHOULD set transaction timeout limits and move expensive operations (external API calls, file processing) outside transaction boundaries.

### Verify

```bash
# Discover the project's test suite location and identify integration tests for API route handlers
find . -type f -name "*.test.*" -o -name "*.spec.*" | grep -E "(route|handler|api)" | head -20

# Execute tests that verify concurrent request handling and transaction isolation
npm test -- --testPathPattern="(route|handler|api)" --testNamePattern="(concurrent|transaction|isolation)" 2>&1 | head -50

# Locate the project's static analysis configuration and run linting rules
find . -maxdepth 2 -type f \( -name ".eslintrc*" -o -name "eslint.config.*" -o -name "tsconfig.json" \) | head -5

# Identify the database query logging configuration
grep -r "transaction\|query.*log\|debug.*db" . --include="*.ts" --include="*.js" --include="*.env*" 2>/dev/null | grep -v node_modules | head -20

# Search for route handlers and verify HTTP method usage
grep -r "router\.\(get\|post\|patch\|delete\|put\)" . --include="*.ts" --include="*.js" 2>/dev/null | grep -v node_modules | head -30

# Verify authorization checks precede database queries
grep -B5 -A5 "findUnique\|findMany\|query" . --include="*.ts" --include="*.js" 2>/dev/null | grep -E "(auth|permission|check|validate)" | head -20
```

**Accept when:**
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error
- Authorization validation executes within the same transaction as subsequent database queries to prevent time-of-check-time-of-use races
- Request-scoped transaction management automatically rolls back on unhandled exceptions and commits on successful response generation

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers MUST be reviewed for compliance with R-CONC-001 through R-CONC-006. Integration tests MUST pass before accepting changes. Static analysis MUST report zero violations of transaction boundary and validation ordering rules.
</enforcement>