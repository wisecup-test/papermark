# Enforce Zod Schema Validation for All Data Access Query Parameters: Validation Failures Terminate

These rules are ALWAYS ACTIVE for all HTTP request handlers that accept query parameters, path parameters, or request body fields used in database operations, and all database query construction code that incorporates external input into where clauses, selection criteria, or data modification operations.

### Rules

- **R-VAL-001** MUST: Validation failures MUST terminate request processing and return appropriate error responses before any database operation is attempted.
- **R-VAL-002** MUST: All external input parameters used in database query predicates and selection criteria MUST be validated against explicit Zod schemas using safeParse or equivalent before reaching the data access layer.
- **R-VAL-003** MUST: Define validation schemas as exported constants in dedicated schema modules organized by domain or feature area to promote reuse and maintainability.
- **R-VAL-004** MUST: When validation fails, log the validation error details for debugging but return sanitized error messages to clients that do not expose internal schema structure or validation logic.
- **R-VAL-005** SHOULD: For array parameters with length constraints, set maximum bounds based on application performance characteristics and database query optimization limits.
- **R-VAL-006** SHOULD: For endpoints handling multiple request types, create separate schemas for each request pattern and use discriminated unions or conditional logic to select the appropriate schema.

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

# Verify validation schemas exist and are used before database operations
grep -r "safeParse\|parse\|validate" . --include="*.ts" --include="*.tsx" | grep -B2 -A2 "findUnique\|findMany\|create\|update\|delete"
```

**Accept when:**
- All API route handlers that accept external input parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that malformed CUID identifiers, out-of-range array lengths, and invalid types are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external input receive parameters typed from validated schemas
- Validation schemas are defined in dedicated modules and exported as constants for reuse
- Sanitized error messages are returned to clients without exposing internal schema structure

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code that falls within the defined scope. Violations must be identified and remediated before merge.
</enforcement>