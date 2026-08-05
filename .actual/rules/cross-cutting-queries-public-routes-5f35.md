# Enforce Explicit Select Projections in Database Queries: Queries Public Routes

These rules are ALWAYS ACTIVE for all database queries in public API route handlers that retrieve entities with sensitive fields, cross authentication or authorization boundaries, or access user data, team data, or multi-tenant resources.

### Rules

- **R-QUERY-001** SHOULD: Queries in public API routes should exclude internal metadata fields such as deletion timestamps, internal identifiers, and audit fields unless explicitly required.
- **R-QUERY-002** SHOULD: Define explicit select clauses before writing queries, listing only fields required for the response contract and authorization logic.
- **R-QUERY-003** SHOULD: Apply projection discipline recursively to nested relations, explicitly selecting fields from each related entity.
- **R-QUERY-004** SHOULD: Document the rationale for including each field in the select clause when necessity is not immediately obvious from surrounding code.
- **R-QUERY-005** SHOULD: Audit all existing queries when adding new fields to entities to determine whether the new field should be included in their projections.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linter or type checker
# to identify database queries without explicit select clauses

# Locate the project's test suite and run integration tests that verify API responses
# do not contain fields marked as sensitive in the schema

# Identify the project's code review checklist and confirm it includes verification that
# new database queries in public API routes include explicit select projections
```

**Accept when:**
- All database queries in public API routes that retrieve entities with sensitive fields include explicit select clauses
- Static analysis or linting passes without warnings about missing select clauses in security-sensitive contexts
- Integration tests confirm that API responses contain only the fields specified in the select projections and do not leak sensitive data
- Code review process specifically checks for explicit select clauses in database queries within API routes

<enforcement>
Clause Code MUST NOT skip or defer verification of explicit select projections in public API route queries. Violations must be caught during code review and static analysis before deployment.
</enforcement>