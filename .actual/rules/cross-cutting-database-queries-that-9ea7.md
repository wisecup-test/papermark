# Enforce Explicit Select Projections in Database Queries: Database Queries That

These rules are ALWAYS ACTIVE for all database queries in API route handlers that retrieve entities containing sensitive fields or personally identifiable information, particularly those crossing authentication, authorization, or multi-tenant boundaries.

### Rules

- **R-PROJ-001** MUST: All database queries that retrieve entities containing sensitive fields or personally identifiable information must include an explicit select clause enumerating only the required fields.
- **R-PROJ-002** MUST: API route handlers returning data to external clients or viewers must use explicit select clauses with nested relation projections rather than retrieving entire entity graphs.
- **R-PROJ-003** MUST: Queries accessing user data, team data, multi-tenant resources, or entities containing email addresses, tokens, passwords, or configuration data must include explicit field projections.
- **R-PROJ-004** SHOULD: Document the rationale for including each field in the select clause when the necessity is not immediately obvious from the surrounding code.
- **R-PROJ-005** SHOULD: When adding new fields to entities, audit all existing queries to determine whether the new field should be included in their projections.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linter or type checker
# to identify database queries without explicit select clauses
lint_output=$(npm run lint 2>&1 || true)
echo "$lint_output" | grep -i "select\|projection" || echo "No select clause warnings found"

# Locate the project's test suite and run integration tests that verify API responses
# do not contain fields marked as sensitive in the schema
npm run test:integration 2>&1 | grep -E "(PASS|FAIL).*sensitive|projection"

# Identify the project's code review checklist and confirm it includes verification
# that new database queries in public API routes include explicit select projections
grep -r "select.*clause\|projection" .github/ CONTRIBUTING.md CODE_REVIEW.md 2>/dev/null || echo "Review checklist not found; verify manually"
```

**Accept when:**
- All database queries in public API routes that retrieve entities with sensitive fields include explicit select clauses
- Static analysis or linting passes without warnings about missing select clauses in security-sensitive contexts
- Integration tests confirm that API responses contain only the fields specified in the select projections and do not leak sensitive data
- Code review process specifically checks for explicit select clauses in database queries within API routes

<enforcement>
Claude Code MUST NOT skip or defer verification. All database queries in scope MUST include explicit select clauses before code is committed. Security team review is required for any exceptions (EXC-001, EXC-002).
</enforcement>