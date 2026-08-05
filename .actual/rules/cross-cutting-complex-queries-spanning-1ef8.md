# Adopt Direct ORM Update Pattern for Dataroom Freeze State Persistence: Complex Queries Spanning

These rules are ALWAYS ACTIVE for all dataroom freeze archive state persistence operations, queries retrieving dataroom metadata with related entities, updates persisting storage references and cryptographic hashes, and temporal queries filtering entity versions or events by timestamp.

### Rules

- **R-ORM-001** MUST: Include both entity identifier and tenant identifier in where clauses for all ORM update operations to enforce multi-tenant isolation at the data access layer.
- **R-ORM-002** MUST: Persist archive metadata (storage location and cryptographic hash) in a single atomic update payload to prevent inconsistent state where one field succeeds and the other fails.
- **R-ORM-003** SHOULD: Use orderBy directives in complex queries spanning multiple relations to ensure deterministic result ordering for version selection and audit trail generation.
- **R-ORM-004** SHOULD: Use include or select directives with explicit field projections rather than fetching entire related entities to minimize data transfer and serialization overhead.
- **R-ORM-005** SHOULD: Apply temporal filtering using comparison operators within the query builder's where clause rather than filtering results in application code to push filtering to the database layer.
- **R-ORM-006** MAY: Use raw SQL queries with parameterized statements only when query complexity exceeds ORM capabilities or when database-specific optimization features are required for performance-critical paths; document rationale explicitly.

### Verify

```bash
# Discover the project's dependency manifest and identify the ORM library and its resolved version
find . -name 'package.json' -o -name 'Gemfile' -o -name 'go.mod' -o -name 'pom.xml' -o -name 'Cargo.toml' | head -1

# Locate the lock artifact to determine exact resolved ORM version
find . -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'Gemfile.lock' -o -name 'go.sum' -o -name 'Cargo.lock' | head -1

# Find dataroom update operations and verify where clauses include both entity and tenant identifiers
grep -r "update\|where" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" . | grep -i dataroom | grep -i freeze

# Identify query patterns using include or select directives
grep -r "include\|select" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" . | grep -i dataroom

# Find temporal filtering logic and verify comparison operators are in query builder
grep -r "createdAt\|viewedAt\|<=\|>=\|<\|>" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" . | grep -i "where\|filter"
```

**Accept when:**
- All dataroom update operations include compound where clauses with both entity identifier (dataroomId) and tenant identifier (teamId)
- Archive metadata updates contain both freezeArchiveUrl and freezeArchiveHash in a single atomic data payload
- Queries for related entities (views, documents, links, agreements) use include or select directives with explicit field projections
- Temporal filters use query builder comparison operators (e.g., v.createdAt <= docView.viewedAt) within where clauses rather than post-query filtering
- Complex queries spanning multiple relations include orderBy directives for deterministic result ordering

<enforcement>
Claude Code MUST NOT skip or defer verification. All ORM update operations MUST be reviewed for multi-tenant where clauses. All archive metadata updates MUST be verified as atomic. All complex queries MUST be inspected for explicit projections and temporal filtering at the query builder layer.
</enforcement>