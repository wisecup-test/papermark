# Adopt Direct ORM Update Pattern for Dataroom Freeze State Persistence: Nested Relation Projections

These rules are ALWAYS ACTIVE for all dataroom freeze archive state persistence operations, queries retrieving dataroom metadata with related entities, updates persisting storage references and cryptographic hashes, and temporal queries filtering entity versions by timestamp.

### Rules

- **R-ORM-001** MUST: Include both entity identifier and tenant identifier in where clauses for all ORM update operations to enforce multi-tenant isolation at the data access layer.
- **R-ORM-002** MUST: Persist archive metadata (storage location and cryptographic hash) in a single atomic update payload to prevent inconsistent state where one field succeeds and the other fails.
- **R-ORM-003** SHOULD: Nested relation projections SHOULD limit selected fields to those required by the workflow to minimize data transfer and serialization overhead.
- **R-ORM-004** MUST: Use comparison operators within the query builder's where clause for temporal filtering rather than filtering results in application code to push filtering to the database layer.
- **R-ORM-005** MUST: Store all timestamps in UTC timezone and configure database connections to use UTC for timezone-aware timestamp comparison operations.
- **R-ORM-006** SHOULD: Use include or select directives with explicit field projections rather than fetching entire related entities in nested queries.

### Verify

```bash
# Discover the project's dependency manifest and identify the ORM library and its resolved version
grep -r "orm\|database\|prisma\|sequelize\|typeorm" package.json yarn.lock pom.xml Gemfile Cargo.toml 2>/dev/null | head -20

# Locate dataroom update operations and verify where clauses include both entity and tenant identifiers
find . -type f \( -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.java" \) -exec grep -l "dataroom.*update\|freeze.*update" {} \; | xargs grep -A 5 "where.*dataroomId\|where.*teamId" 2>/dev/null

# Identify query patterns using include or select directives
find . -type f \( -name "*.ts" -o -name "*.js" \) -exec grep -l "include\|select" {} \; | xargs grep -B 2 -A 3 "include.*select\|select.*include" 2>/dev/null

# Find temporal filtering logic and verify comparison operators are in query builder
find . -type f \( -name "*.ts" -o -name "*.js" \) -exec grep -l "createdAt\|viewedAt\|timestamp" {} \; | xargs grep -B 2 -A 2 "where.*<=\|where.*>=\|where.*<\|where.*>" 2>/dev/null

# Verify archive metadata updates contain both freezeArchiveUrl and freezeArchiveHash
find . -type f \( -name "*.ts" -o -name "*.js" \) -exec grep -l "freezeArchive" {} \; | xargs grep -B 3 -A 3 "freezeArchiveUrl.*freezeArchiveHash\|freezeArchiveHash.*freezeArchiveUrl" 2>/dev/null
```

**Accept when:**
- All dataroom update operations include compound where clauses with both entity identifier and tenant identifier
- Archive metadata updates contain both storage location (freezeArchiveUrl) and cryptographic hash (freezeArchiveHash) in a single atomic data payload
- Queries for related entities use include or select directives with explicit field projections
- Temporal filters use query builder comparison operators within where clauses rather than post-query filtering
- All timestamps are stored and compared in UTC timezone
- Nested projections explicitly list required fields rather than fetching entire related entities

<enforcement>
Claude Code MUST NOT skip or defer verification. All ORM update operations MUST be reviewed for multi-tenant isolation compliance. All archive metadata updates MUST be verified as atomic operations. All temporal queries MUST use database-layer filtering via query builder operators.
</enforcement>