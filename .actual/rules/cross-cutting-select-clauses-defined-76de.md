# Enforce Explicit Select Projections in Database Queries: Select Clauses Defined

These rules are ALWAYS ACTIVE for all database queries in API route handlers that retrieve entities with sensitive fields, cross authentication or authorization boundaries, or return data to external clients.

### Rules

- **R-SEL-001** SHOULD: Select clauses should be defined near the query invocation rather than abstracted into distant helper functions to maintain visibility of data access patterns.

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
Claude Code MUST NOT skip or defer verification. Code review MUST block merge until select clauses are added to queries in public API routes. Static analysis failures in continuous integration MUST prevent deployment.
</enforcement>