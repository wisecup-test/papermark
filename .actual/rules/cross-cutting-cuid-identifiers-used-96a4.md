# Enforce Zod Schema Validation for All Data Access Query Parameters: Cuid Identifiers Used

These rules are ALWAYS ACTIVE for all HTTP request handlers that accept query parameters, path parameters, or request body fields used in database operations, and all database query construction code that incorporates external input into where clauses, selection criteria, or data modification operations.

### Rules

- **R-CUID-001** MUST: CUID identifiers used in where clauses MUST be validated as valid CUID format strings before use in queries.
- **R-CUID-002** MUST: All API route handlers accepting external input parameters demonstrate schema validation with safeParse or equivalent before database queries execute.
- **R-CUID-003** MUST: Validation schemas MUST be defined as exported constants in dedicated schema modules organized by domain or feature area.
- **R-CUID-004** MUST: When validation fails, log validation error details for debugging but return sanitized error messages to clients that do not expose internal schema structure.
- **R-CUID-005** SHOULD: For array parameters with length constraints, set maximum bounds based on application performance characteristics and database query optimization limits.
- **R-CUID-006** SHOULD: For endpoints handling multiple request types, create separate schemas for each request pattern and use discriminated unions or conditional logic to select the appropriate schema.

### Verify

```bash
# Discover the project's test suite configuration and locate integration tests for API route handlers
# Execute tests that verify validation rejection behavior for malformed input
find . -name "*.test.*" -o -name "*.spec.*" | head -20

# Discover the project's static analysis or linting configuration
# Run type checking to verify that database query parameters are derived from validated schema types
grep -r "typecheck\|tsc\|type-check" . --include="package.json" --include="tsconfig.json" --include="*.config.*"

# Discover the project's code search or grep capabilities
# Search for database query construction patterns and verify each query using external input has corresponding validation logic
grep -r "findUnique\|findMany\|create\|update\|delete" . --include="*.ts" --include="*.tsx" | grep -E "where|data" | head -20

# Verify safeParse usage in route handlers
grep -r "safeParse" . --include="*.ts" --include="*.tsx" | head -20

# Verify schema definitions exist in dedicated modules
find . -path "*/schema*" -o -path "*/schemas*" | grep -E "\.ts$" | head -20
```

**Accept when:**
- All API route handlers that accept external input parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that malformed CUID identifiers, out-of-range array lengths, and invalid types are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external input receive parameters typed from validated schemas
- Validation schemas are organized in dedicated modules and exported as constants
- Error responses sanitize validation details and do not expose internal schema structure

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-CUID-001 through R-CUID-006 must be verified before accepting code changes that introduce or modify database query handlers accepting external input.
</enforcement>