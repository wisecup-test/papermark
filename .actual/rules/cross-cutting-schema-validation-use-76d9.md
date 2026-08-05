# Enforce Zod Schema Validation for All Data Access Query Parameters: Schema Validation Use

These rules are ALWAYS ACTIVE for all HTTP request handlers that accept query parameters, path parameters, or request body fields used in database operations, and all database query construction code that incorporates external input into where clauses, selection criteria, or data modification operations.

### Rules

- **R-ZOD-001** MUST: Schema validation MUST use safeParse or equivalent non-throwing validation methods that return structured success/failure results before any database query execution.
- **R-ZOD-002** MUST: All API route handlers accepting external input parameters used in database operations MUST validate those parameters against explicit Zod schemas before reaching the data access layer.
- **R-ZOD-003** MUST: Database query where clauses, selection criteria, and data modification operations MUST NOT incorporate external input without prior schema validation.
- **R-ZOD-004** SHOULD: Define validation schemas as exported constants in dedicated schema modules organized by domain or feature area to promote reuse and maintainability.
- **R-ZOD-005** SHOULD: For endpoints handling multiple request types, create separate schemas for each request pattern and use discriminated unions or conditional logic to select the appropriate schema.
- **R-ZOD-006** SHOULD: When validation fails, log the validation error details for debugging but return sanitized error messages to clients that do not expose internal schema structure or validation logic.
- **R-ZOD-007** SHOULD: For array parameters with length constraints, set maximum bounds based on application performance characteristics and database query optimization limits.

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

# Verify safeParse usage in validation patterns
grep -r "safeParse" . --include="*.ts" --include="*.tsx" | wc -l
```

**Accept when:**
- All API route handlers that accept external input parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that malformed CUID identifiers, out-of-range array lengths, and invalid types are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external input receive parameters typed from validated schemas
- Validation schemas are organized in dedicated modules and reused across endpoints
- Validation failures return sanitized error messages without exposing internal schema structure

<enforcement>
Claude Code MUST NOT skip or defer verification. All new database query code MUST include corresponding validation logic before merge. Static type checking MUST pass. Security scanning for unvalidated database queries MUST show zero violations.
</enforcement>