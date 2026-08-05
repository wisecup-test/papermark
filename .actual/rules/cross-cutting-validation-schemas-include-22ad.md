# Enforce Zod Schema Validation for All Data Access Query Parameters: Validation Schemas Include

These rules are ALWAYS ACTIVE for all HTTP request handlers that accept query parameters, path parameters, or request body fields used in database operations, and all database query construction code that incorporates external input into where clauses, selection criteria, or data modification operations.

### Rules

- **R-VAL-001** MUST: Define validation schemas as exported constants in dedicated schema modules organized by domain or feature area before any database query uses external input parameters.
- **R-VAL-002** MUST: Apply schema validation using safeParse or equivalent non-throwing methods to all external input parameters before they are used in database query predicates, where clauses, or selection criteria.
- **R-VAL-003** MUST: Check validation results and return appropriate HTTP error responses (e.g., 400 Bad Request) when validation fails, without proceeding to database operations.
- **R-VAL-004** MUST: Ensure all database query where clauses using external input receive parameters typed from validated schemas, verified by static type checking.
- **R-VAL-005** SHOULD: Include additional constraints beyond type checking in validation schemas such as string length limits, numeric ranges, or custom refinements for business logic validation.
- **R-VAL-006** SHOULD: Log validation error details for debugging purposes while returning sanitized error messages to clients that do not expose internal schema structure or validation logic.
- **R-VAL-007** SHOULD: For array parameters with length constraints, set maximum bounds based on application performance characteristics and database query optimization limits.
- **R-VAL-008** MAY: Create separate schemas for endpoints handling multiple request types and use discriminated unions or conditional logic to select the appropriate schema.

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

# Verify validation schemas exist in dedicated modules
find . -path "*/schema*" -o -path "*/schemas*" -type f -name "*.ts" | head -20

# Verify safeParse usage in route handlers
grep -r "safeParse" . --include="*.ts" --include="*.tsx" | head -20
```

**Accept when:**
- All API route handlers that accept external input parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that malformed CUID identifiers, out-of-range array lengths, and invalid types are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external input receive parameters typed from validated schemas
- Validation schemas are defined in dedicated, reusable modules organized by domain or feature area
- Validation failures return sanitized error messages without exposing internal schema structure

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-VAL-001 through R-VAL-008 must be checked against the codebase before accepting any changes to data access query parameter handling.
</enforcement>