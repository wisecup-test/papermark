# Enforce HTTP Method-Based Concurrency Control in API Routes: Database Mutations Include

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

### Rules

- **R-DBMUT-001** MUST: All database mutations MUST include ownership or team membership verification in the WHERE clause to prevent unauthorized modifications.

### Verify

```bash
# Discover the project's test suite location and execute integration tests that verify HTTP method handling, input validation, and authorization for API routes
find . -type f -name '*test*' -o -name '*spec*' | head -5

# Discover the project's static analysis configuration and execute type checking to verify schema validation is applied before database operations
find . -type f \( -name 'tsconfig.json' -o -name '.eslintrc*' -o -name 'jest.config.*' \) | head -5

# Discover the project's security testing tools and execute authorization tests that attempt unauthorized access across all HTTP methods
grep -r 'security\|auth\|test' package.json 2>/dev/null | head -10
```

**Accept when:**
- All API route handlers explicitly validate HTTP methods and return appropriate error responses for unsupported methods
- Schema validation is applied to all request inputs before database operations are performed, with validation failures returning structured error responses
- Authorization checks are present for all database mutations and verify ownership or team membership in query predicates
- Integration tests demonstrate that unauthorized requests are rejected with appropriate HTTP status codes for each HTTP method

<enforcement>
Claude Code MUST NOT skip or defer verification of database mutation authorization predicates. All mutations must include explicit ownership or team membership checks in WHERE clauses before code is approved.
</enforcement>