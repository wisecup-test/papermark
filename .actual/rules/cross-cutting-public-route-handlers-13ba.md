# Enforce Zod Schema Validation for All Public API Route Inputs: Public Route Handlers

These rules are ALWAYS ACTIVE for all public API route handlers accepting external HTTP requests with untrusted input in POST, GET, PATCH, and DELETE endpoints.

### Rules

- **R-ZVAL-001** MUST: All public API route handlers MUST validate request bodies using schema validation before accessing any properties or executing business logic.
- **R-ZVAL-002** MUST: Request body parsing and deserialization logic in POST, PATCH, and PUT endpoints MUST include schema validation before database queries or ORM operations.
- **R-ZVAL-003** MUST: Query parameter extraction and processing in GET and DELETE endpoints MUST be validated against explicit schemas before use in business logic.
- **R-ZVAL-004** MUST: Path parameter extraction used in resource identification and authorization checks MUST be validated before accessing request data.
- **R-ZVAL-005** MUST: Input validation MUST precede all database queries, ORM operations, or external service calls.
- **R-ZVAL-006** SHOULD: Define validation schemas as exported constants in dedicated schema modules to enable reuse across route handlers and test suites.
- **R-ZVAL-007** SHOULD: Structure route handlers to perform validation as the first operation after extracting request data, returning early with error responses before executing authorization checks.
- **R-ZVAL-008** SHOULD: For routes accepting multiple input sources, validate each source independently and combine validation results to provide comprehensive error feedback.

### Verify

```bash
# Discover the project's static analysis configuration and execute linting/type-checking
# to verify all route handlers include validation calls before business logic
grep -r "safeParse\|parse\|validate" --include="*.ts" --include="*.js" src/routes/ | grep -v "test\|spec"

# Locate the project's test suite and execute integration tests targeting API route handlers
# to verify validation behavior with invalid inputs
npm test -- --testPathPattern="(route|api|handler)" --testNamePattern="(validation|invalid|error)"

# Scan route handler implementations for validation method invocations
# to confirm coverage across all public endpoints
grep -l "export.*handler\|export.*route" src/routes/*.ts | xargs grep -L "safeParse\|parse\|validate" | head -20
```

**Accept when:**
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation
- Validation schemas are defined in dedicated modules and imported into route handlers
- Validation occurs as the first operation in route handlers before authorization checks

<enforcement>
Claude Code MUST NOT skip or defer verification. All public route handlers MUST be scanned for validation coverage before approving changes. Pull requests adding or modifying route handlers without validation MUST be blocked.
</enforcement>