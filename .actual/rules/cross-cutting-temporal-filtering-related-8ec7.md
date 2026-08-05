# Adopt Direct ORM Update Pattern for Dataroom Freeze State Persistence: Temporal Filtering Related

These rules are ALWAYS ACTIVE for all dataroom freeze archive state persistence operations, queries retrieving dataroom metadata with temporal constraints, and updates that persist storage references and cryptographic hashes to dataroom records.

### Rules

- **R-FREEZE-001** MUST: Temporal filtering on related entities SHALL use comparison operators against timestamp fields within the query builder rather than post-query filtering in application code.
- **R-FREEZE-002** MUST: All dataroom update operations include compound where clauses with both entity identifier and tenant identifier to enforce multi-tenant isolation at the data access layer.
- **R-FREEZE-003** MUST: Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload to prevent consistency windows where one field is persisted but the other is missing.
- **R-FREEZE-004** SHOULD: Queries for related entities use include or select directives with explicit field projections rather than fetching entire entities to minimize data transfer and serialization overhead.
- **R-FREEZE-005** SHOULD: When designing nested projections with include and select directives, explicitly list only required fields and use temporal filters within query builder methods rather than post-query filtering.

### Verify

```bash
# Discover the project's dependency manifest and identify the ORM library and its resolved version from the lock artifact
find . -name 'package.json' -o -name 'Gemfile.lock' -o -name 'go.sum' -o -name 'Cargo.lock' | head -1

# Locate the module containing dataroom update operations and verify that where clauses include both entity and tenant identifiers
grep -r "dataroom.*update\|freeze.*update" --include="*.ts" --include="*.js" --include="*.go" --include="*.rb" | grep -v node_modules | head -20

# Identify query patterns using include or select directives and confirm they specify explicit field projections
grep -r "include\|select" --include="*.ts" --include="*.js" --include="*.go" --include="*.rb" | grep -E "(dataroom|freeze|archive)" | grep -v node_modules | head -20

# Find temporal filtering logic and verify comparison operators are applied within query builder methods
grep -r "createdAt\|updatedAt\|timestamp" --include="*.ts" --include="*.js" --include="*.go" --include="*.rb" | grep -E "(where|filter|<=|>=|<|>)" | grep -v node_modules | head -20
```

**Accept when:**
- All dataroom update operations include compound where clauses with both entity identifier and tenant identifier
- Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload
- Queries for related entities use include or select directives with explicit field projections
- Temporal filters use query builder comparison operators rather than post-query filtering in application code
- No update operations exist without multi-tenant where clauses
- All temporal filtering logic is embedded in query builder methods, not in application-level filtering loops

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for dataroom freeze archive operations. Violations must be corrected before code review approval.
</enforcement>