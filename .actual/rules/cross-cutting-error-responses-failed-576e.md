# Enforce Zod Schema Validation for All Public API Route Inputs: Error Responses Failed

These rules are ALWAYS ACTIVE for all public API route handlers accepting external HTTP requests with untrusted input, including POST, GET, PATCH, and DELETE endpoints that process request bodies, query parameters, and path parameters.

### Rules

- **R-VAL-001** SHOULD: Error responses from failed validation SHOULD include sufficient detail for API consumers to correct malformed requests without exposing internal implementation details.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting or type-checking command
# to verify all route handlers include validation calls before business logic
grep -r "safeParse\|parse\|validate" --include="*.ts" --include="*.js" src/routes/ | grep -v node_modules

# Locate the project's test suite directory structure and execute the integration test command
# targeting API route handlers to verify validation behavior with invalid inputs
npm test -- --testPathPattern="(route|api|endpoint)" --testNamePattern="(validation|invalid|error)"

# Identify the project's code search capability and scan route handler implementations
# for validation method invocations to confirm coverage across all public endpoints
grep -r "schema\.parse\|schema\.safeParse\|validate(" --include="*.ts" --include="*.js" src/ | wc -l
```

**Accept when:**
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation
- Error response messages provide actionable feedback for API consumers without exposing internal implementation details, database schemas, or internal identifiers

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers must be scanned for validation coverage, and error response structures must be reviewed to ensure they meet the SHOULD requirement for sufficient detail without information disclosure.
</enforcement>