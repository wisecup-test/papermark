# Enforce Zod Schema Validation for All Data Access Query Parameters: Validation Schemas Defined

These rules are ALWAYS ACTIVE for all HTTP request handlers that accept query parameters, path parameters, or request body fields used in database operations, and all database query construction code that incorporates external input into where clauses, selection criteria, or data modification operations.

### Rules

- **R-VAL-001** SHOULD: Validation schemas SHOULD be defined as reusable named constants or exported schema objects rather than inline anonymous schemas.
- **R-VAL-002** MUST: All API route handlers that accept external input parameters MUST demonstrate schema validation with safeParse or equivalent before database queries.
- **R-VAL-003** MUST: All database query where clauses using external input MUST receive parameters typed from validated schemas.
- **R-VAL-004** SHOULD: Validation schemas SHOULD be organized in dedicated schema modules by domain or feature area to promote reuse and maintainability.
- **R-VAL-005** SHOULD: When validation fails, SHOULD log validation error details for debugging but return sanitized error messages to clients that do not expose internal schema structure.
- **R-VAL-006** SHOULD: For array parameters with length constraints, SHOULD set maximum bounds based on application performance characteristics and database query optimization limits.

### Verify

```bash
# Discover the project's test suite configuration and locate integration tests for API route handlers
# Execute tests that verify validation rejection behavior for malformed input
find . -type f -name '*.test.*' -o -name '*.spec.*' | head -20

# Discover the project's static analysis or linting configuration
# Run type checking to verify that database query parameters are derived from validated schema types
grep -r "typecheck\|tsc\|type-check" . --include="package.json" --include="tsconfig.json" --include="*.config.*"

# Discover the project's code search capabilities
# Search for database query construction patterns and verify each query using external input has corresponding validation logic
grep -r "findUnique\|findMany\|create\|update\|delete" . --include="*.ts" --include="*.tsx" | grep -E "where|data" | head -20

# Verify validation schemas are defined as named constants or exported objects
grep -r "export.*const.*Schema\|export.*const.*schema" . --include="*.ts" --include="*.tsx" | head -20

# Verify safeParse usage in route handlers
grep -r "safeParse" . --include="*.ts" --include="*.tsx" -B 2 -A 2 | head -30
```

**Accept when:**
- All API route handlers that accept external input parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that malformed CUID identifiers, out-of-range array lengths, and invalid types are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external input receive parameters typed from validated schemas
- Validation schemas are defined as exported constants or named objects in dedicated schema modules, not inline
- Validation error logging includes details for debugging while client responses contain sanitized messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code that falls within the stated scope. Violations MUST be identified and remediated before merge.
</enforcement>