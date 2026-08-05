# Enforce Explicit Select Projections in Database Queries: Select Projections Scoped

These rules are ALWAYS ACTIVE for all API route handlers that return data to external clients, database queries that retrieve user data, team data, or multi-tenant resources, and operations that cross authentication or authorization boundaries.

### Rules

- **R-SEL-001** MUST: Select projections must be scoped to the minimum set of fields necessary for the operation's business logic and response contract.

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

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review blocks merge until select clauses are added to queries in public API routes. Static analysis failures in continuous integration prevent deployment.
</enforcement>