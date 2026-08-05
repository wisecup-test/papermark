# Enforce Zod Schema Validation for All Public API Route Inputs: Validation Schemas Define

These rules are ALWAYS ACTIVE for all public API route handlers accepting external HTTP requests with untrusted input, including POST, GET, PATCH, and DELETE endpoints that process request bodies, query parameters, and path parameters before business logic execution.

### Rules

- **R-VAL-001** MUST: Validation schemas MUST define constraints including string formats, numeric ranges, array bounds, and required fields matching business requirements.
- **R-VAL-002** MUST: All HTTP route handlers exposed as public API endpoints accepting external requests MUST validate inputs against explicit schemas before processing.
- **R-VAL-003** MUST: Request body parsing and deserialization logic in POST, PATCH, and PUT endpoints MUST include validation preceding database queries or business logic execution.
- **R-VAL-004** MUST: Query parameter extraction and processing in GET and DELETE endpoints MUST be validated before use in authorization checks or data access.
- **R-VAL-005** MUST: Path parameter extraction used in resource identification and authorization checks MUST be validated against defined schemas.
- **R-VAL-006** MUST: Validation schemas MUST be defined as exported constants in dedicated schema modules to enable reuse across route handlers and test suites.
- **R-VAL-007** MUST: Route handlers MUST perform validation as the first operation after extracting request data, returning early with error responses before executing authorization checks or database queries.
- **R-VAL-008** MUST: For routes accepting multiple input sources, each source MUST be validated independently with combined validation results providing comprehensive error feedback.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting or type-checking command
# to verify all route handlers include validation calls before business logic

# Locate the project's test suite directory structure and execute the integration test command
# targeting API route handlers to verify validation behavior with invalid inputs

# Identify the project's code search capability and scan route handler implementations
# for validation method invocations to confirm coverage across all public endpoints
grep -r "safeParse\|parse\|validate" --include="*.ts" --include="*.js" src/routes/ || echo "No validation calls found"
```

**Accept when:**
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation
- Validation schemas are defined in dedicated modules and imported into route handlers
- Validation occurs as the first operation in route handlers before authorization or data access logic

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API route handlers MUST be scanned for validation coverage before approving changes. Pull requests adding or modifying route handlers without validation MUST be blocked.
</enforcement>