# Enforce Zod Schema Validation for All Data Access Query Parameters: Array Parameters Used

These rules are ALWAYS ACTIVE for all HTTP request handlers that accept external input parameters used in database queries, including query parameters, path parameters, and request body fields incorporated into where clauses, selection criteria, or data modification operations.

### Rules

- **R-ARRAY-001** MUST: Array parameters used in database queries MUST validate both array structure and individual element constraints including minimum and maximum length bounds.

### Verify

```bash
# Discover the project's test suite configuration and locate integration tests for API route handlers
# Execute tests that verify validation rejection behavior for malformed input
find . -name "*.test.*" -o -name "*.spec.*" | head -20

# Discover the project's static analysis or linting configuration
# Run type checking to verify that database query parameters are derived from validated schema types
grep -r "zod\|schema" --include="*.ts" --include="*.tsx" | grep -i "array\|length" | head -20

# Discover the project's code search capabilities
# Search for database query construction patterns and verify each query using external input has corresponding validation logic
grep -r "findMany\|findUnique\|where:" --include="*.ts" --include="*.tsx" | grep -v "node_modules" | head -20
```

**Accept when:**
- All API route handlers that accept external array parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that out-of-range array lengths and invalid array structures are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external array input receive parameters typed from validated schemas
- Array validation schemas explicitly define minimum and maximum length bounds appropriate to application performance and database query optimization limits

<enforcement>
Claude Code MUST NOT skip or defer verification. All array parameters used in database queries MUST be validated before reaching the data access layer.
</enforcement>