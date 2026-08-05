# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Progress Metadata Updated

These rules are ALWAYS ACTIVE for all archive operations, long-running dataroom freeze tasks, batch processing workflows, and any operation involving streaming compression, multipart uploads, or progress tracking through metadata updates.

### Rules

- **R-ASYNC-001** MUST: Progress metadata MUST be updated at defined checkpoints during archive operations to provide incremental feedback on operation status.
- **R-ASYNC-002** MUST: Archive operations that process multiple documents and produce compressed artifacts MUST use asynchronous task orchestration patterns rather than synchronous request handlers.
- **R-ASYNC-003** MUST: Operations involving multipart uploads to object storage with streaming hash computation MUST use transform streams to compute cryptographic hashes incrementally during archive creation and upload.
- **R-ASYNC-004** MUST: All event emitters and streams in asynchronous workflows MUST register error handlers to propagate failures through the workflow, ensuring cleanup operations such as upload abortion execute before task failure.
- **R-ASYNC-005** MUST: Batch processing operations MUST partition documents by cumulative size rather than count alone to ensure batches remain within memory and processing time constraints.
- **R-ASYNC-006** SHOULD: Archive task functions SHOULD accept payload objects containing dataroom identifiers and team context, and coordinate multiple asynchronous workflows including document retrieval, batch processing, streaming compression, hash computation, and multipart upload.
- **R-ASYNC-007** SHOULD: Cleanup operations such as upload abortion and resource release SHOULD execute before task failure is reported.
- **R-ASYNC-008** MAY: Operations with proven completion times under 10 seconds may use synchronous implementations with documented justification and technical lead approval.

### Verify

```bash
# Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
find . -type f -name '*.json' -o -name '*.yaml' -o -name '*.yml' | xargs grep -l 'task\|orchestration\|archive' | head -5

# Locate the test suite covering archive task execution
find . -type f \( -name '*test*' -o -name '*spec*' \) | xargs grep -l 'archive\|batch.*process\|progress.*update' | head -10

# Verify tests exist for batch processing, progress updates, error handling, and cleanup on failure
grep -r 'describe\|test\|it(' . --include='*test*' --include='*spec*' | grep -E 'batch|progress|error|cleanup' | head -20

# Identify multipart upload and hash computation implementations
find . -type f \( -name '*.js' -o -name '*.ts' \) | xargs grep -l 'multipart\|hash.*stream\|transform.*stream' | head -10

# Execute archive task tests
grep -r 'test.*archive\|describe.*archive' . --include='*test*' --include='*spec*' -A 5 | head -30
```

**Accept when:**
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly
- All asynchronous operations register error handlers on event emitters and streams
- Batch processing partitions documents by cumulative size with meaningful progress increments
- Transform streams are used for incremental hash computation during archive creation and upload

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review verification that new long-running operations follow asynchronous task patterns with streaming interfaces is mandatory. Automated test coverage requirements for asynchronous workflows including error handling and cleanup paths must be confirmed. Architecture review for operations exceeding expected synchronous execution time must verify task orchestration adoption. Pull requests implementing long-running operations as synchronous endpoints MUST be rejected with guidance to adopt task orchestration patterns. Operations that buffer entire archives in memory rather than using streaming interfaces MUST be flagged in code review for refactoring. Missing error handlers on asynchronous operations MUST be identified and required to be added before merge.
</enforcement>