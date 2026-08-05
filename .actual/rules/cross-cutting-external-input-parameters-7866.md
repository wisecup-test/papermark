# Enforce Zod Schema Validation for All Data Access Query Parameters: External Input Parameters

These rules are ALWAYS ACTIVE for all HTTP request handlers that accept external input parameters (query parameters, path parameters, or request body fields) used in database query predicates, where clauses, or selection criteria.

### Rules

- **R-ZVAL-001** MUST: All external input parameters used in database query predicates, where clauses, or selection criteria MUST be validated against an explicit schema before the query is constructed or executed.
- **R-ZVAL-002** MUST: Validation schemas MUST be defined as exported constants in dedicated schema modules organized by domain or feature area to promote reuse and maintainability.
- **R-ZVAL-003** MUST: For endpoints handling multiple request types, create separate schemas for each request pattern and use discriminated unions or conditional logic to select the appropriate schema.
- **R-ZVAL-004** MUST: When validation fails, log the validation error details for debugging but return sanitized error messages to clients that do not expose internal schema structure or validation logic.
- **R-ZVAL-005** SHOULD: For array parameters with length constraints, set maximum bounds based on application performance characteristics and database query optimization limits.
- **R-ZVAL-006** SHOULD: Use non-throwing safeParse methods to allow request handlers to distinguish between validation failures and runtime errors, enabling appropriate HTTP status codes and error messages.

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
grep -r "findUnique\|findMany\|create\|update\|delete" . --include="*.ts" --include="*.tsx" | grep -E "where|select" | head -20

# Verify validation schemas exist and are exported
find . -path "*/schema*" -o -path "*/schemas*" -type f -name "*.ts" | head -20

# Search for safeParse usage patterns
grep -r "safeParse" . --include="*.ts" --include="*.tsx" | head -20
```

**Accept when:**
- All API route handlers that accept external input parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that malformed CUID identifiers, out-of-range array lengths, and invalid types are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external input receive parameters typed from validated schemas
- Validation schemas are defined in dedicated modules and exported as constants
- Error responses sanitize validation details and do not expose internal schema structure

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory and must be verified before accepting code changes. Static type checking and integration tests are required to confirm compliance.
</enforcement>