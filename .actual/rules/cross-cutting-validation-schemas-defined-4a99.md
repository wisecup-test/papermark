# Enforce Zod Schema Validation for All Public API Route Inputs: Validation Schemas Defined

These rules are ALWAYS ACTIVE for all public API route handlers accepting external HTTP requests with untrusted input, including POST, GET, PATCH, and DELETE endpoints that process request bodies, query parameters, and path parameters.

### Rules

- **R-VAL-001** SHOULD: Validation schemas SHOULD be defined as reusable schema objects separate from route handler implementations to enable testing and reuse.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting or type-checking command to verify all route handlers include validation calls before business logic
# (Exact command depends on project build tool and linter configuration)

# Locate the project's test suite directory structure and execute the integration test command targeting API route handlers to verify validation behavior with invalid inputs
# (Exact command depends on project test runner configuration)

# Identify the project's code search or grep capability and scan route handler implementations for validation method invocations to confirm coverage across all public endpoints
grep -r "safeParse\|parse\|validate" --include="*.ts" --include="*.js" src/routes/
```

**Accept when:**
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API route handlers must be scanned to confirm validation schemas are defined separately and applied before business logic execution.
</enforcement>