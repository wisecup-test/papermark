# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Long Running Archive

These rules are ALWAYS ACTIVE for all long-running archive operations that involve document collection, batch processing, streaming compression, hash computation, multipart uploads, and progress tracking through metadata updates.

### Rules

- **R-ASYNC-001** MUST: Long-running archive operations MUST be implemented as asynchronous task functions that coordinate multiple concurrent workflows rather than synchronous request handlers.
- **R-ASYNC-002** MUST: Archive operations MUST use transform streams to compute cryptographic hashes incrementally during archive creation and upload, piping archive output through hash computation transforms before passing to upload streams to avoid multiple passes over data.
- **R-ASYNC-003** MUST: Batch processing MUST partition documents by cumulative size rather than count alone, ensuring batches remain within memory and processing time constraints while providing meaningful progress increments.
- **R-ASYNC-004** MUST: Error handlers MUST be registered on all event emitters and streams to propagate failures through the workflow, ensuring cleanup operations such as upload abortion execute before task failure.
- **R-ASYNC-005** MUST: Archive task functions MUST accept payload objects containing dataroom identifiers and team context, and coordinate multiple asynchronous workflows including document retrieval, batch processing, streaming compression, hash computation, and multipart upload.
- **R-ASYNC-006** SHOULD: Progress metadata updates SHOULD be treated as non-fatal warnings if they fail independently of the archive operation, with timeout-based progress estimation as fallback and final operation status always persisted.

### Verify

```bash
# Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
find . -type f -name '*.ts' -o -name '*.js' | xargs grep -l 'archive.*task\|task.*archive' | head -5

# Locate the test suite covering archive task execution
find . -type f \( -name '*.test.ts' -o -name '*.spec.ts' -o -name '*.test.js' -o -name '*.spec.js' \) | xargs grep -l 'archive.*task\|batch.*process\|progress.*update' | head -5

# Verify tests exist for batch processing, progress updates, error handling, and cleanup on failure
grep -r 'describe\|it\|test' --include='*.test.ts' --include='*.spec.ts' | grep -E 'batch|progress|error|cleanup' | head -10

# Identify integration test infrastructure
find . -type f -path '*/test/*' -o -path '*/tests/*' | grep -E 'integration|e2e' | head -5

# Verify streaming interfaces are used for archive generation
grep -r 'createReadStream\|createWriteStream\|Transform\|pipe' --include='*.ts' --include='*.js' | grep -i archive | head -5
```

**Accept when:**
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly
- All archive operations use streaming interfaces rather than buffering entire archives in memory
- Error handlers are present on all event emitters and streams in archive task implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. All archive operations must be reviewed for compliance with R-ASYNC-001 through R-ASYNC-006 before merge. Operations that buffer entire archives in memory or lack error handlers on streams are violations requiring remediation.
</enforcement>