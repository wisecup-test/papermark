# Enforce HTTP Method-Based Concurrency Control in API Routes: Database Queries That

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

### Rules

- **R-CONC-001** SHOULD: Database queries that check existence before mutation SHOULD be combined into atomic operations where the database system supports it.

### Verify

```bash
# Discover the project's test suite location and execute integration tests that verify HTTP method handling, input validation, and authorization for API routes
find . -type f -name '*test*' -o -name '*spec*' | head -5

# Discover the project's static analysis configuration and execute type checking to verify schema validation is applied before database operations
find . -type f \( -name '.eslintrc*' -o -name 'tsconfig.json' -o -name 'pyproject.toml' -o -name '.flake8' \) | head -5

# Discover the project's security testing tools and execute authorization tests that attempt unauthorized access across all HTTP methods
find . -type f -name '*security*' -o -name '*auth*test*' | head -5
```

**Accept when:**
- All API route handlers explicitly validate HTTP methods and return appropriate error responses for unsupported methods
- Schema validation is applied to all request inputs before database operations are performed, with validation failures returning structured error responses
- Authorization checks are present for all database mutations and verify ownership or team membership in query predicates
- Integration tests demonstrate that unauthorized requests are rejected with appropriate HTTP status codes for each HTTP method
- Database queries include authorization predicates in WHERE clauses rather than performing separate authorization checks after data retrieval
- Validation schemas are defined adjacent to their corresponding HTTP method handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers must be audited for compliance with R-CONC-001 before code is merged.
</enforcement>