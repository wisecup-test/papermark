# Adopt Direct ORM Update Pattern for Dataroom Freeze State Persistence: Workflows Use Orm

These rules are ALWAYS ACTIVE for all dataroom freeze archive state persistence operations, queries retrieving dataroom metadata for archive generation, updates persisting storage references and cryptographic hashes, and temporal queries filtering entity versions by timestamp.

### Rules

- **R-ORM-001** MAY: Workflows MAY use the ORM findUnique method with nested select projections to retrieve specific related entity fields when only a single record is expected.
- **R-ORM-002** MUST: All dataroom update operations include compound where clauses with both entity identifier and tenant identifier to enforce multi-tenant isolation at the data access layer.
- **R-ORM-003** MUST: Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload to prevent inconsistent state.
- **R-ORM-004** MUST: Queries for related entities use include or select directives with explicit field projections rather than fetching entire entities.
- **R-ORM-005** MUST: Temporal filtering logic use comparison operators within the query builder's where clause rather than filtering results in application code.
- **R-ORM-006** MUST: Store all timestamps in UTC timezone and configure database connections to use UTC for timezone-aware comparison operations.

### Verify

```bash
# Discover the project's dependency manifest and identify the ORM library and its resolved version
find . -name 'package.json' -o -name 'Gemfile' -o -name 'go.mod' -o -name 'pom.xml' | head -1

# Locate the lock artifact to determine exact resolved ORM version
find . -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'Gemfile.lock' -o -name 'go.sum' | head -1

# Find modules containing dataroom update operations
grep -r "update.*where" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" | grep -i dataroom

# Verify where clauses include both entity and tenant identifiers
grep -r "where.*dataroomId.*teamId\|where.*teamId.*dataroomId" --include="*.ts" --include="*.js"

# Identify query patterns using include or select directives
grep -r "include\|select" --include="*.ts" --include="*.js" | grep -i dataroom

# Find temporal filtering logic and verify comparison operators in query builder
grep -r "createdAt\|viewedAt" --include="*.ts" --include="*.js" | grep -E "<=|>=|<|>"
```

**Accept when:**
- All dataroom update operations include compound where clauses with both entity identifier and tenant identifier
- Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload
- Queries for related entities use include or select directives with explicit field projections
- Temporal filters use query builder comparison operators rather than post-query filtering
- All timestamps are stored and compared in UTC timezone
- ORM version is pinned to a specific range in the dependency manifest

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory and must be verified before code is accepted. Rules marked MAY are permissible patterns. Violations trigger build failures and pull request blocks until corrected.
</enforcement>