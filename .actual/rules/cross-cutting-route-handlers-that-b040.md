# Enforce Zod Schema Validation for All API Route Input Parameters: Route Handlers That

These rules are ALWAYS ACTIVE for all API route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

### Rules

- **R-ZOD-001** MUST: All API route handlers that accept external input MUST validate request bodies, query parameters, path parameters, and cookies using schema validation before processing.
- **R-ZOD-002** MUST: Validation of identifiers used in database queries including primary keys and foreign keys MUST occur before any database query executes.
- **R-ZOD-003** MUST: Use safe parsing methods that return success or failure results and check the success property before accessing validated data to ensure type safety.
- **R-ZOD-004** MUST: Return appropriate HTTP status codes for validation failures with error messages that describe what validation failed without exposing internal implementation details.
- **R-ZOD-005** MUST: For endpoints that accept arrays, always specify maximum length constraints to prevent resource exhaustion from oversized payloads.
- **R-ZOD-006** SHOULD: Define validation schemas in dedicated schema modules organized by domain or feature area to promote reuse across related endpoints.

### Verify

```bash
# Discover and run the project's test suite covering API route validation logic
find . -name "*.test.*" -o -name "*.spec.*" | head -5

# Discover the project's static analysis or linting configuration
find . -name ".eslintrc*" -o -name "eslint.config.*" -o -name "tsconfig.json" | head -5

# Search for validation schema definitions in the codebase
find . -path ./node_modules -prune -o -type f -name "*schema*" -print | grep -E "\.(ts|js)$" | head -10

# Identify API route handlers and check for validation patterns
grep -r "safeParse\|parse\|validate" --include="*.ts" --include="*.js" | grep -E "route|handler|api" | head -10

# Check for database queries and their position relative to validation
grep -r "db\.\|query\|execute" --include="*.ts" --include="*.js" | grep -E "route|handler|api" | head -10
```

**Accept when:**
- All API route handlers that accept external input include schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes
- Validation schemas are defined in dedicated modules and reused across related endpoints
- Safe parsing methods are used with explicit success/failure result checking
- Array inputs include maximum length constraints in validation schemas
- HTTP error responses for validation failures do not expose internal implementation details

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API route handlers MUST include schema validation before processing external input. Pull requests without input validation are blocked until validation is added.
</enforcement>