# Enforce Zod Schema Validation for All Data Access Query Parameters: Complex Request Bodies

These rules are ALWAYS ACTIVE for all HTTP request handlers that accept query parameters, path parameters, or request body fields used in database operations, and all database query construction code that incorporates external input into where clauses, selection criteria, or data modification operations.

### Rules

- **R-ZOD-CRB-001** SHOULD: Complex request bodies containing multiple validated fields SHOULD use object schemas that validate all fields atomically.

### Verify

```bash
# Discover the project's test suite configuration and locate integration tests for API route handlers
# Execute tests that verify validation rejection behavior for malformed input
find . -name '*.test.*' -o -name '*.spec.*' | head -20

# Discover the project's static analysis or linting configuration
# Run type checking to verify that database query parameters are derived from validated schema types
grep -r "typecheck\|tsc\|type-check" . --include="package.json" --include="tsconfig.json" --include="*.config.*"

# Discover the project's code search capabilities
# Search for database query construction patterns and verify each query using external input has corresponding validation logic
grep -r "findUnique\|findMany\|create\|update\|delete" . --include="*.ts" --include="*.tsx" | grep -E "where|data" | head -20
```

**Accept when:**
- All API route handlers that accept external input parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that malformed CUID identifiers, out-of-range array lengths, and invalid types are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external input receive parameters typed from validated schemas

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands MUST be executed and their results reviewed before accepting code that modifies API route handlers or database query construction.
</enforcement>