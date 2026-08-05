# Enforce Zod Schema Validation for All Public API Route Inputs: Route Handlers Perform

These rules are ALWAYS ACTIVE for all public API route handlers accepting external HTTP requests with untrusted input in POST, GET, PATCH, and DELETE endpoints.

### Rules

- **R-ZVAL-001** MUST: All public API route handlers validate request inputs (body, query parameters, path parameters) against explicit schemas before executing business logic or database queries.
- **R-ZVAL-002** MUST: Schema validation must occur as the first operation after extracting request data, with early return of error responses before authorization checks or database access.
- **R-ZVAL-003** MUST: Validation schemas must be defined as exported constants in dedicated schema modules to enable reuse across route handlers and test suites.
- **R-ZVAL-004** MUST: Routes accepting multiple input sources must validate each source independently and combine validation results to provide comprehensive error feedback identifying all invalid fields.
- **R-ZVAL-005** MAY: Route handlers MAY perform additional business-logic validation after schema validation passes for constraints requiring database lookups or complex rules.
- **R-ZVAL-006** MUST NOT: Route handlers must not access request body properties, query parameters, or path parameters without prior schema validation.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting or type-checking command
# to verify all route handlers include validation calls before business logic

# Locate the project's test suite directory structure and execute the integration test command
# targeting API route handlers to verify validation behavior with invalid inputs

# Identify the project's code search capability and scan route handler implementations
# for validation method invocations to confirm coverage across all public endpoints
grep -r "safeParse\|parse\|validate" --include="*.ts" --include="*.js" src/routes/ || echo "No validation found"
```

**Accept when:**
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation
- Validation schemas are centralized in dedicated modules and imported by route handlers
- Error responses provide actionable feedback without exposing internal implementation details

<enforcement>
Claude Code MUST NOT skip or defer verification of schema validation coverage across all public API route handlers. Violations must be identified and remediated before merge.
</enforcement>