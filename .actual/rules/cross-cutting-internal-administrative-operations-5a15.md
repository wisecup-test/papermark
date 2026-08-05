# Enforce Explicit Select Projections in Database Queries: Internal Administrative Operations

These rules are ALWAYS ACTIVE for all database query code in API route handlers that retrieve user data, team data, multi-tenant resources, or entities containing sensitive fields (email addresses, tokens, passwords, configuration data), particularly those crossing authentication or authorization boundaries.

### Rules

- **R-PROJ-001** MUST: Include explicit select clauses in all database queries within public API route handlers that retrieve entities with sensitive fields.
- **R-PROJ-002** MUST: Apply projection discipline recursively to nested relations, explicitly selecting only required fields from each related entity.
- **R-PROJ-003** SHOULD: Document the rationale for including each field in the select clause when necessity is not immediately obvious from surrounding code.
- **R-PROJ-004** SHOULD: Audit all existing queries when adding new fields to entities to determine whether the new field should be included in their projections.
- **R-PROJ-005** MAY: Internal administrative operations may retrieve full entities when operating within trusted boundaries and when audit logging captures the access (EXC-001, EXC-002).

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
Claude Code MUST NOT skip or defer verification. All database queries in public API routes must include explicit select clauses before code is approved. Static analysis failures in continuous integration prevent deployment. Security team conducts periodic audits of data access patterns and files remediation tickets for violations.
</enforcement>