# Enforce Zod Schema Validation for All Public API Route Inputs: Path Parameters Query

These rules are ALWAYS ACTIVE for all public API route handlers accepting external HTTP requests with path parameters and query parameters used in authorization decisions or database queries.

### Rules

- **R-ZVAL-001** MUST: All path parameters and query parameters used in authorization decisions or database queries MUST be validated against schemas defining expected types and formats before processing.
- **R-ZVAL-002** MUST: Validation schemas MUST be defined as exported constants in dedicated schema modules to enable reuse across route handlers and test suites.
- **R-ZVAL-003** MUST: Route handlers MUST perform validation as the first operation after extracting request data, returning early with error responses before executing authorization checks or database queries.
- **R-ZVAL-004** MUST: For routes accepting multiple input sources, each source MUST be validated independently with validation results combined to provide comprehensive error feedback identifying all invalid fields in a single response.
- **R-ZVAL-005** SHOULD: Validation error messages SHOULD provide actionable feedback without revealing database schemas, internal identifiers, or existence of resources.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting or type-checking command
# to verify all route handlers include validation calls before business logic
grep -r "safeParse\|parse\|validate" --include="*.ts" --include="*.js" src/routes/ | grep -v node_modules

# Locate the project's test suite directory structure and execute integration tests
# targeting API route handlers to verify validation behavior with invalid inputs
npm test -- --testPathPattern="routes|api" --testNamePattern="validation|invalid"

# Scan route handler implementations for validation method invocations
# to confirm coverage across all public endpoints
grep -l "export.*route\|export.*handler" src/routes/*.ts | xargs grep -L "safeParse\|parse\|validate" | head -20
```

**Accept when:**
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation
- Validation schemas are defined in dedicated modules and imported into route handlers
- Validation occurs as the first operation in route handlers before authorization or database access

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers must be scanned for validation coverage before accepting changes to public API endpoints.
</enforcement>