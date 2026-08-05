# Enforce HTTP Method-Based Concurrency Control in API Routes: Database Query Operations

These rules are ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

### Rules

- **R-CCDB-001** MUST: Database query operations MUST use explicit field selection to minimize data exposure and prevent over-fetching of sensitive fields.

### Verify

```bash
# Discover the project's test suite location and execute integration tests that verify HTTP method handling, input validation, and authorization for API routes
# Discover the project's static analysis configuration and execute type checking to verify schema validation is applied before database operations
# Discover the project's security testing tools and execute authorization tests that attempt unauthorized access across all HTTP methods
```

**Accept when:**
- All API route handlers explicitly validate HTTP methods and return appropriate error responses for unsupported methods
- Schema validation is applied to all request inputs before database operations are performed, with validation failures returning structured error responses
- Authorization checks are present for all database mutations and verify ownership or team membership in query predicates
- Integration tests demonstrate that unauthorized requests are rejected with appropriate HTTP status codes for each HTTP method
- Database queries consistently use explicit field selection in SELECT clauses rather than wildcard selection

<enforcement>
Claude Code MUST NOT skip or defer verification. All API route handlers must be inspected for explicit field selection, method-specific validation schemas, and authorization predicates in database query WHERE clauses before code is approved.
</enforcement>