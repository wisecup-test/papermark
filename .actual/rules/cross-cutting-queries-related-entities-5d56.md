# Adopt Direct ORM Update Pattern for Dataroom Freeze State Persistence: Queries Related Entities

These rules are ALWAYS ACTIVE for all dataroom freeze archive state persistence operations, queries retrieving dataroom metadata with related entities, and updates that persist storage references and cryptographic hashes to dataroom records.

### Rules

- **R-FREEZE-001** MUST: Queries for related entities SHALL use the ORM include or select directives to declare required relations and projections explicitly rather than performing separate queries and manual joins.
- **R-FREEZE-002** MUST: All dataroom update operations include compound where clauses with both entity identifier and tenant identifier to enforce multi-tenant isolation at the data access layer.
- **R-FREEZE-003** MUST: Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload to ensure consistency between related metadata fields.
- **R-FREEZE-004** MUST: Temporal filtering logic use comparison operators within the query builder's where clause rather than filtering results in application code to push filtering to database layer.
- **R-FREEZE-005** SHOULD: When designing nested projections with include and select directives, explicitly list only required fields rather than fetching entire related entities to minimize data transfer and serialization overhead.

### Verify

```bash
# Discover the project's dependency manifest and identify the ORM library and its resolved version from the lock artifact
find . -name 'package.json' -o -name 'Gemfile' -o -name 'requirements.txt' -o -name 'go.mod' -o -name 'Cargo.toml' | head -1

# Locate the module containing dataroom update operations and verify that where clauses include both entity and tenant identifiers
grep -r "dataroom.*update\|freeze.*update" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" . | grep -v node_modules | head -20

# Identify query patterns using include or select directives and confirm they specify explicit field projections
grep -r "include\|select" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" . | grep -E "(dataroom|freeze|archive)" | grep -v node_modules | head -20

# Find temporal filtering logic and verify comparison operators are applied within query builder methods
grep -r "createdAt\|viewedAt\|timestamp" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" . | grep -E "(where|filter|<=|>=|<|>)" | grep -v node_modules | head -20
```

**Accept when:**
- All dataroom update operations include compound where clauses with both entity identifier and tenant identifier
- Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload
- Queries for related entities use include or select directives with explicit field projections
- Temporal filters use query builder comparison operators rather than post-query filtering
- No separate queries are performed for related entities that could be joined in a single ORM call

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for dataroom freeze archive operations.
</enforcement>