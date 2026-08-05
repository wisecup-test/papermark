# Enforce HTTP Method-Based Concurrency Control in API Routes: Routes Implement Request

These rules are ALWAYS ACTIVE for all HTTP API route handlers that process external requests and perform database operations with method-specific validation, authentication, and authorization patterns.

### Rules

- **R-ROUTE-001** MUST: All API route handlers explicitly validate HTTP methods and return appropriate error responses for unsupported methods.
- **R-ROUTE-002** MUST: Schema validation is applied to all request inputs (body and query parameters) before database operations are performed, with validation failures returning structured error responses.
- **R-ROUTE-003** MUST: Authorization checks are present for all database mutations and verify ownership or team membership in query predicates rather than post-retrieval filtering.
- **R-ROUTE-004** MUST: Database queries include explicit field selection to minimize data exposure and enforce authorization boundaries at the data access layer.
- **R-ROUTE-005** SHOULD: Organize route handlers by grouping HTTP method handlers within a single route file, with early method validation and explicit error responses for unsupported methods.
- **R-ROUTE-006** SHOULD: Define validation schemas adjacent to their corresponding HTTP method handlers to maintain clear association between validation rules and handler logic.
- **R-ROUTE-007** SHOULD: Implement consistent error response formatting across all routes using a shared error handler that maps validation failures, authorization denials, and database errors to appropriate HTTP status codes.
- **R-ROUTE-008** MAY: Routes MAY implement request-specific logging for security-sensitive operations such as authentication failures or authorization denials.

### Verify

```bash
# Discover and execute the project's integration test suite
# Verify HTTP method handling, input validation, and authorization for API routes
find . -type f -name '*test*' -o -name '*spec*' | grep -E '(route|api|handler)' | head -1

# Discover and execute static analysis / type checking
# Verify schema validation is applied before database operations
find . -name 'tsconfig.json' -o -name '.eslintrc*' -o -name 'pyproject.toml' | head -1

# Discover security testing tools
# Execute authorization tests that attempt unauthorized access across all HTTP methods
find . -type f -name '*security*' -o -name '*auth*test*' | head -1

# Verify all route files contain explicit HTTP method validation
grep -r "method.*===\|switch.*method\|if.*method" --include='*.ts' --include='*.js' --include='*.py' | wc -l

# Verify schema validation is applied before database queries
grep -r "schema\.parse\|validate\|zod\|joi" --include='*.ts' --include='*.js' --include='*.py' | wc -l

# Verify authorization predicates in database queries
grep -r "WHERE.*user\|WHERE.*team\|filter.*owner" --include='*.ts' --include='*.js' --include='*.py' | wc -l
```

**Accept when:**
- All API route handlers explicitly validate HTTP methods and return appropriate error responses for unsupported methods
- Schema validation is applied to all request inputs before database operations are performed, with validation failures returning structured error responses
- Authorization checks are present for all database mutations and verify ownership or team membership in query predicates
- Integration tests demonstrate that unauthorized requests are rejected with appropriate HTTP status codes for each HTTP method
- Database queries include explicit field selection and authorization predicates in WHERE clauses
- Error response formatting is consistent across all routes using a shared error handler

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory and must be verified before code is considered compliant. Rules marked SHOULD represent strong recommendations that should be followed unless explicitly justified. Rules marked MAY are optional enhancements.
</enforcement>