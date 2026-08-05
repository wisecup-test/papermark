# Adopt Direct ORM Update Pattern for Dataroom Freeze State Persistence: Archive Metadata Updates

These rules are ALWAYS ACTIVE for all dataroom freeze archive state persistence operations, queries retrieving dataroom metadata with related entities, and updates that persist storage references and cryptographic hashes to dataroom records.

### Rules

- **R-ARCHIVE-001** MUST: Archive metadata updates SHALL include both the storage location reference and cryptographic hash in a single atomic data payload to maintain consistency between location and content verification.
- **R-ARCHIVE-002** MUST: All dataroom update operations include compound where clauses with both entity identifier (dataroomId) and tenant identifier (teamId) to enforce multi-tenant isolation at the data access layer.
- **R-ARCHIVE-003** MUST: Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload, never as separate sequential updates.
- **R-ARCHIVE-004** SHOULD: Queries for related entities (views, documents, links, agreements) use include or select directives with explicit field projections rather than fetching entire entities.
- **R-ARCHIVE-005** SHOULD: Temporal filtering logic use comparison operators within the query builder's where clause rather than filtering results in application code to push filtering to database layer.
- **R-ARCHIVE-006** SHOULD: When designing nested projections with include and select directives, explicitly list only required fields to minimize data transfer and serialization overhead.

### Verify

```bash
# Discover the project's dependency manifest and identify the ORM library and its resolved version
find . -name 'package.json' -o -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' -o -name 'Gemfile.lock' -o -name 'go.sum' -o -name 'Cargo.lock' | head -5

# Locate the module containing dataroom update operations
grep -r "freezeArchiveUrl\|freezeArchiveHash" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" | grep -E "(update|where)" | head -20

# Verify that where clauses include both entity and tenant identifiers
grep -r "where.*dataroomId.*teamId\|where.*teamId.*dataroomId" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" | head -20

# Identify query patterns using include or select directives
grep -r "include\s*:\|select\s*:" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" | grep -v node_modules | head -20

# Find temporal filtering logic and verify comparison operators
grep -r "createdAt\|viewedAt\|<=\|>=\|<\|>" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" | grep -E "(where|filter)" | head -20
```

**Accept when:**
- All dataroom update operations include compound where clauses with both entity identifier and tenant identifier
- Archive metadata updates contain both storage location (freezeArchiveUrl) and cryptographic hash (freezeArchiveHash) in a single atomic data payload
- Queries for related entities use include or select directives with explicit field projections
- Temporal filters use query builder comparison operators within where clauses rather than post-query filtering
- No separate sequential update calls exist for storage location and hash fields

<enforcement>
Claude Code MUST NOT skip or defer verification. All archive metadata persistence operations MUST be reviewed to confirm atomic updates with compound multi-tenant where clauses before code is accepted.
</enforcement>