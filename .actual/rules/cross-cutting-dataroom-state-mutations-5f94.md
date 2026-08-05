# Adopt Direct ORM Update Pattern for Dataroom Freeze State Persistence: Dataroom State Mutations

These rules are ALWAYS ACTIVE for all dataroom state mutation operations, archive metadata persistence, and related entity queries within the dataroom freeze archive workflow.

### Rules

- **R-DATAROOM-001** MUST: All dataroom state mutations that persist archive artifacts SHALL use the ORM update method with explicit where clauses specifying both entity identifier and team identifier to enforce multi-tenant isolation.

### Verify

```bash
# Discover the project's dependency manifest and identify the ORM library and its resolved version from the lock artifact
find . -name 'package.json' -o -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' -o -name 'Gemfile.lock' -o -name 'go.sum' -o -name 'Cargo.lock' | head -5

# Locate the module containing dataroom update operations and verify that where clauses include both entity and tenant identifiers
grep -r "dataroom.*update\|freezeArchive" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" | grep -E "where.*teamId|where.*dataroomId" | head -20

# Identify query patterns using include or select directives and confirm they specify explicit field projections
grep -r "include\|select" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" | grep -E "dataroom|freeze" | head -20

# Find temporal filtering logic and verify comparison operators are applied within query builder methods
grep -r "createdAt\|viewedAt\|<=\|>=" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" | grep -E "where|query" | head -20
```

**Accept when:**
- All dataroom update operations include compound where clauses with both entity identifier and team identifier
- Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload
- Queries for related entities use include or select directives with explicit field projections and temporal filters use query builder comparison operators
- No update operations exist that modify dataroom state without multi-tenant where clause enforcement

<enforcement>
Claude Code MUST NOT skip or defer verification. All dataroom state mutations MUST be reviewed to confirm multi-tenant isolation enforcement at the data access layer before code is committed.
</enforcement>