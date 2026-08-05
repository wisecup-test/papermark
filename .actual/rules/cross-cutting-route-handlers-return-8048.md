# Enforce Zod Schema Validation for All Public API Route Inputs: Route Handlers Return

These rules are ALWAYS ACTIVE for all HTTP route handlers exposed as public API endpoints accepting external requests, including POST, PATCH, PUT, GET, and DELETE endpoints that process request bodies, query parameters, and path parameters.

### Rules

- **R-ROUTE-001** MUST: Route handlers MUST return appropriate HTTP error status codes when validation fails, preventing execution of business logic with invalid inputs.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting or type-checking command
# to verify all route handlers include validation calls before business logic
grep -r "safeParse\|parse\|validate" --include="*.ts" --include="*.js" src/routes/ | head -20

# Locate the project's test suite directory structure and execute the integration test command
# targeting API route handlers to verify validation behavior with invalid inputs
find . -name "*.test.ts" -o -name "*.spec.ts" | grep -i route | head -10

# Identify the project's code search capability and scan route handler implementations
# for validation method invocations to confirm coverage across all public endpoints
grep -r "if (!.*\.success)" --include="*.ts" --include="*.js" src/routes/
```

**Accept when:**
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers must be scanned for validation coverage before accepting changes to public API endpoints.
</enforcement>