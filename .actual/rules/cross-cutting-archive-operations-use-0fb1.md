# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Archive Operations Use

These rules are ALWAYS ACTIVE for all archive operations that involve document collection, batch processing, streaming compression, hash computation, multipart uploads, or long-running workflows requiring progress tracking through metadata updates.

### Rules

- **R-ASYNC-ARCHIVE-001** MUST: Archive operations MUST use streaming interfaces for hash computation and upload operations to avoid loading entire archives into memory.
- **R-ASYNC-ARCHIVE-002** MUST: Archive task functions MUST accept payload objects containing dataroom identifiers and team context, and coordinate multiple asynchronous workflows including document retrieval, batch processing, streaming compression, hash computation, and multipart upload.
- **R-ASYNC-ARCHIVE-003** MUST: Transform streams MUST be used to compute cryptographic hashes incrementally during archive creation and upload, piping archive output through hash computation transforms before passing to upload streams to avoid multiple passes over data.
- **R-ASYNC-ARCHIVE-004** MUST: Batch processing MUST partition documents by cumulative size rather than count alone, ensuring batches remain within memory and processing time constraints while providing meaningful progress increments.
- **R-ASYNC-ARCHIVE-005** MUST: Error handlers MUST be registered on all event emitters and streams to propagate failures through the workflow, ensuring cleanup operations such as upload abortion execute before task failure.
- **R-ASYNC-ARCHIVE-006** MUST: Progress metadata updates MUST occur at defined checkpoints (0.05, 0.1, per-batch) to provide incremental user feedback during long-running archive operations.
- **R-ASYNC-ARCHIVE-007** SHOULD: Retry logic with exponential backoff SHOULD be implemented to handle task orchestration infrastructure failures or quota limits.
- **R-ASYNC-ARCHIVE-008** SHOULD: Periodic cleanup jobs SHOULD be implemented for orphaned resources such as incomplete multipart uploads or intermediate files.
- **R-ASYNC-ARCHIVE-009** MAY: Operations with proven completion times under 10 seconds MAY use synchronous implementations with documented justification and technical lead approval.

### Verify

```bash
# Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
find . -type f -name '*.json' -o -name '*.yaml' -o -name '*.yml' | xargs grep -l 'task\|orchestration' | head -5

# Locate the test suite covering archive task execution
find . -type f \( -name '*archive*.test.*' -o -name '*archive*.spec.*' \) | head -10

# Verify tests exist for batch processing, progress updates, error handling, and cleanup
grep -r 'batch\|progress\|error.*handler\|cleanup' --include='*.test.*' --include='*.spec.*' | grep -i archive | head -20

# Identify the project's integration test infrastructure
find . -type d -name 'integration' -o -name 'e2e' -o -name '__tests__' | head -5

# Execute archive task tests
grep -r 'describe\|test\|it(' --include='*.test.*' --include='*.spec.*' | grep -i 'archive' | head -15
```

**Accept when:**
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly
- Code review verification confirms new long-running operations follow asynchronous task patterns with streaming interfaces
- Automated test coverage exists for asynchronous workflows including error handling and cleanup paths
- Architecture review confirms operations exceeding expected synchronous execution time use task orchestration patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. All archive operations MUST be reviewed against these rules before merge. Operations that buffer entire archives in memory rather than using streaming interfaces MUST be flagged for refactoring. Missing error handlers on asynchronous operations MUST be identified and required to be added before merge.
</enforcement>