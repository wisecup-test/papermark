# Adopt Structured Logging with Contextual Metadata for Asynchronous Archive Operations: Structured Log Metadata

These rules are ALWAYS ACTIVE for all asynchronous archive operations, multi-stage document processing workflows, batch processing operations with progress tracking, operations involving cloud storage uploads and lambda invocations, and CSV generation and export operations.

### Rules

- **R-STRUCT-LOG-001** SHOULD: Structured log metadata SHOULD include computed metrics such as row counts, file counts, and size conversions to facilitate operational analysis without post-processing.
- **R-STRUCT-LOG-002** MUST: All logging statements in multi-stage asynchronous operations MUST include structured metadata objects with relevant operational context.
- **R-STRUCT-LOG-003** MUST: Progress tracking mechanism MUST update both numeric progress indicators and status text at key operation milestones.
- **R-STRUCT-LOG-004** MUST: Batch processing operations MUST emit structured metadata including batch number, total batches, file counts, and size metrics at batch start and completion.
- **R-STRUCT-LOG-005** SHOULD: Define standard metadata schema conventions for common operation types including required fields for identifiers, counts, sizes, and timestamps to ensure consistency across the codebase.
- **R-STRUCT-LOG-006** MUST: Progress metadata updates MUST include error handling to ensure updates succeed or trigger alerts, preventing silent failures that leave progress tracking out of sync.
- **R-STRUCT-LOG-007** MUST: For stream-based operations, error handlers MUST log structured error context and trigger cleanup operations such as aborting partial uploads to prevent resource leaks.

### Verify

```bash
# Discover and execute the project's static analysis configuration to verify that logging statements
# in multi-stage operations include structured metadata objects.
find . -name '.eslintrc*' -o -name 'pylintrc' -o -name '.flake8' -o -name 'tox.ini' | head -1 | xargs -I {} sh -c 'echo "Found config: {}"; cat {}'

# Discover and run the project's test suite to verify that progress metadata updates occur at
# expected operation milestones and include both numeric progress and status text.
find . -name 'package.json' -o -name 'pytest.ini' -o -name 'setup.py' | head -1 | xargs -I {} sh -c 'echo "Found manifest: {}"; grep -E "test|spec" {}'

# Discover and execute the project's log validation tooling to verify that batch processing
# operations emit batch-level metadata including sequence numbers, counts, and size metrics.
find . -name '*log*' -type f \( -name '*.js' -o -name '*.py' -o -name '*.ts' \) | grep -i valid | head -5

# Grep for structured metadata patterns in archive operation files
grep -r "metadata.*=.*{" --include="*.js" --include="*.ts" --include="*.py" | grep -E "(batch|progress|archive)" | head -10
```

**Accept when:**
- All logging statements in multi-stage asynchronous operations include structured metadata objects with relevant operational context.
- Progress tracking mechanism updates both numeric progress indicators and status text at key operation milestones.
- Batch processing operations emit structured metadata including batch number, total batches, file counts, and size metrics at batch start and completion.
- Standard metadata schema conventions are defined and documented for common operation types.
- Progress metadata update error handling is implemented with validation to ensure updates succeed or trigger alerts.
- Stream-based operation error handlers log structured error context and trigger cleanup operations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis rules MUST flag logging statements missing structured metadata in designated operation types. Code review MUST verify that new logging statements include structured metadata objects before merge approval. Integration tests MUST validate that progress metadata updates occur at expected milestones with correct values.
</enforcement>