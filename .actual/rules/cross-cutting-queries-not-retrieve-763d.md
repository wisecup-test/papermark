# Enforce Explicit Select Projections in Database Queries: Queries Not Retrieve

These rules are ALWAYS ACTIVE for all database queries in API route handlers that retrieve entities with sensitive fields or cross tenant boundaries.

### Rules

- **R-QUERY-001** MUST_NOT: Queries must not retrieve entire entity graphs without select clauses when the entity contains fields marked as sensitive or when the query crosses tenant boundaries.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linter or type checker
# to identify database queries without explicit select clauses
find . -name '.eslintrc*' -o -name 'tsconfig.json' -o -name '.prettierrc*' | head -1

# Locate the project's test suite and run integration tests that verify API responses
# do not contain fields marked as sensitive in the schema
find . -type f -name '*.test.*' -o -name '*.spec.*' | grep -i 'api\|route\|response' | head -5

# Identify the project's code review checklist and confirm it includes verification
# that new database queries in public API routes include explicit select projections
find . -name 'CONTRIBUTING.md' -o -name '.github/PULL_REQUEST_TEMPLATE.md' -o -name 'CODE_REVIEW.md'
```

**Accept when:**
- All database queries in public API routes that retrieve entities with sensitive fields include explicit select clauses
- Static analysis or linting passes without warnings about missing select clauses in security-sensitive contexts
- Integration tests confirm that API responses contain only the fields specified in the select projections and do not leak sensitive data
- Code review process specifically checks for explicit select clauses in database queries within API routes

<enforcement>
Claude Code MUST NOT skip or defer verification. All database queries in public API routes must include explicit select clauses before code is approved.
</enforcement>