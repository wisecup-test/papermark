# Enforce HTTP Method-Based Concurrency Control in API Routes: Error Responses Use

These rules are ALWAYS ACTIVE for all HTTP API route handlers that accept external requests and perform database operations with method-specific validation and authorization.

### Rules

- **R-HTTP-001** SHOULD: Error responses SHOULD use consistent HTTP status codes and structured error messages without exposing internal implementation details.

### Verify

```bash
# Discover the project's test suite location and execute integration tests that verify HTTP method handling, input validation, and authorization for API routes
find . -type f -name '*test*' -o -name '*spec*' | grep -E '(integration|e2e)' | head -5

# Discover the project's static analysis configuration and execute type checking to verify schema validation is applied before database operations
find . -type f \( -name 'tsconfig.json' -o -name '.eslintrc*' -o -name 'jest.config.*' \) | head -5

# Discover the project's security testing tools and execute authorization tests that attempt unauthorized access across all HTTP methods
find . -type f -name '*security*' -o -name '*auth*test*' | head -5
```

**Accept when:**
- All API route handlers explicitly validate HTTP methods and return appropriate error responses for unsupported methods
- Schema validation is applied to all request inputs before database operations are performed, with validation failures returning structured error responses
- Authorization checks are present for all database mutations and verify ownership or team membership in query predicates
- Integration tests demonstrate that unauthorized requests are rejected with appropriate HTTP status codes for each HTTP method
- Error responses use consistent HTTP status codes (4xx for client errors, 5xx for server errors) without leaking stack traces or internal implementation details

<enforcement>
Claude Code MUST NOT skip or defer verification. All error response patterns in API route handlers MUST be audited against R-HTTP-001 before code approval.
</enforcement>