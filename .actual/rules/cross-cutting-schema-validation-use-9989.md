# Enforce Zod Schema Validation for All Public API Route Inputs: Schema Validation Use

These rules are ALWAYS ACTIVE for all public API route handlers accepting external HTTP requests with untrusted input, including POST, GET, PATCH, and DELETE endpoints that process request bodies, query parameters, and path parameters.

### Rules

- **R-SCHEMA-001** MUST: Schema validation MUST use safeParse or equivalent non-throwing methods to enable explicit error handling and appropriate HTTP error responses.
- **R-SCHEMA-002** MUST: All HTTP route handlers exposed as public API endpoints accepting external requests MUST validate inputs against explicit schemas before processing.
- **R-SCHEMA-003** MUST: Request body parsing and deserialization logic in POST, PATCH, and PUT endpoints MUST include schema validation.
- **R-SCHEMA-004** MUST: Query parameter extraction and processing in GET and DELETE endpoints MUST include schema validation.
- **R-SCHEMA-005** MUST: Path parameter extraction used in resource identification and authorization checks MUST include schema validation.
- **R-SCHEMA-006** MUST: Input validation MUST precede database queries, ORM operations, or external service calls.
- **R-SCHEMA-007** SHOULD: Define validation schemas as exported constants in dedicated schema modules to enable reuse across route handlers and test suites.
- **R-SCHEMA-008** SHOULD: Structure route handlers to perform validation as the first operation after extracting request data, returning early with error responses before executing authorization checks or database queries.
- **R-SCHEMA-009** SHOULD: For routes accepting multiple input sources, validate each source independently and combine validation results to provide comprehensive error feedback identifying all invalid fields in a single response.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting or type-checking command
# to verify all route handlers include validation calls before business logic

# Locate the project's test suite directory structure and execute the integration test command
# targeting API route handlers to verify validation behavior with invalid inputs

# Identify the project's code search or grep capability and scan route handler implementations
# for validation method invocations to confirm coverage across all public endpoints
```

**Accept when:**
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution.
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints.
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation.
- Validation schemas are defined in dedicated modules and imported into route handlers.
- Validation occurs as the first operation in route handlers before authorization checks or database access.

<enforcement>
Claude Code MUST NOT skip or defer verification of schema validation coverage across all public API route handlers. Violations blocking merge require remediation before acceptance.
</enforcement>