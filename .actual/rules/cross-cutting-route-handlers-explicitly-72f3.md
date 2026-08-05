# Enforce HTTP Method-Based Concurrency Control in API Routes: Route Handlers Explicitly

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

### Rules

- **R-HTTP-001** MUST: All API route handlers MUST explicitly declare which HTTP methods they accept and reject requests with unsupported methods before processing.
- **R-HTTP-002** MUST: Define validation schemas adjacent to their corresponding HTTP method handlers to maintain clear association between validation rules and handler logic.
- **R-HTTP-003** MUST: Structure database queries to include authorization predicates in WHERE clauses rather than performing separate authorization checks after data retrieval.
- **R-HTTP-004** MUST: Apply schema validation to all request inputs before database operations are performed, with validation failures returning structured error responses.
- **R-HTTP-005** MUST: Implement consistent error response formatting across all routes using a shared error handler that maps validation failures, authorization denials, and database errors to appropriate HTTP status codes.
- **R-HTTP-006** SHOULD: Use database-level constraints and atomic upsert operations to mitigate race conditions between existence checks and subsequent mutations.
- **R-HTTP-007** SHOULD: Implement automated security testing that verifies authorization enforcement for each HTTP method.
- **R-HTTP-008** SHOULD: Sanitize validation error messages before returning to clients while logging detailed errors internally.

### Verify

```bash
# Discover and execute integration tests for HTTP method handling, input validation, and authorization
find . -type f -name '*test*' -o -name '*spec*' | head -5

# Discover and execute static analysis for schema validation before database operations
find . -name '.eslintrc*' -o -name 'tsconfig.json' -o -name 'pyproject.toml' | head -3

# Discover and execute security testing tools for authorization bypass detection
find . -type f -name '*security*' -o -name '*audit*' | head -5

# Verify HTTP method validation in route handlers
grep -r "method.*===\|method.*==\|allowedMethods\|supportedMethods" --include="*.ts" --include="*.js" | head -10

# Verify authorization predicates in database queries
grep -r "WHERE.*userId\|WHERE.*teamId\|WHERE.*owner" --include="*.ts" --include="*.js" | head -10
```

**Accept when:**
- All API route handlers explicitly validate HTTP methods and return appropriate error responses for unsupported methods
- Schema validation is applied to all request inputs before database operations are performed, with validation failures returning structured error responses
- Authorization checks are present for all database mutations and verify ownership or team membership in query predicates
- Integration tests demonstrate that unauthorized requests are rejected with appropriate HTTP status codes for each HTTP method
- Database queries include explicit field selection and authorization predicates in WHERE clauses
- Error responses are consistently formatted across all routes with appropriate HTTP status codes

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers MUST be audited for explicit HTTP method validation, schema validation application, and authorization predicate inclusion before code is considered compliant.
</enforcement>