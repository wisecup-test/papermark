# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Asynchronous Helper Functions

These rules are ALWAYS ACTIVE for all archive generation tasks, long-running document processing workflows, multipart upload operations, and batch processing operations that coordinate multiple asynchronous data retrieval and transformation steps.

### Rules

- **R-ASYNC-001** SHOULD: Asynchronous helper functions for CSV generation and intermediate file cleanup SHOULD be extracted as separate composable units.
- **R-ASYNC-002** MUST: Archive operations MUST use task functions that accept payload objects containing dataroom identifiers and team context, coordinating multiple asynchronous workflows including document retrieval, batch processing, streaming compression, hash computation, and multipart upload.
- **R-ASYNC-003** MUST: Transform streams MUST be used to compute cryptographic hashes incrementally during archive creation and upload, piping archive output through hash computation transforms before passing to upload streams to avoid multiple passes over data.
- **R-ASYNC-004** MUST: Batch processing MUST partition documents by cumulative size rather than count alone, ensuring batches remain within memory and processing time constraints while providing meaningful progress increments.
- **R-ASYNC-005** MUST: Error handlers MUST be registered on all event emitters and streams to propagate failures through the workflow, ensuring cleanup operations such as upload abortion execute before task failure.
- **R-ASYNC-006** MUST: Progress tracking MUST be implemented through incremental metadata updates at checkpoints during long-running operations to provide user feedback on archive creation status.
- **R-ASYNC-007** MUST: All asynchronous operations MUST register error handlers that trigger cleanup, and periodic cleanup jobs MUST be implemented for orphaned resources.
- **R-ASYNC-008** MAY: Operations with proven completion times under 10 seconds may use synchronous implementations with documented justification.
- **R-ASYNC-009** MAY: Prototype or experimental features may defer full asynchronous implementation with explicit technical debt tracking.

### Verify

```bash
# Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
find . -type f -name '*.json' -o -name '*.yaml' -o -name '*.yml' | xargs grep -l 'task\|orchestration' | head -5

# Locate the test suite covering archive task execution
find . -type f \( -name '*archive*.test.*' -o -name '*archive*.spec.*' -o -name 'test-*archive*' \) | head -10

# Verify tests exist for batch processing, progress updates, error handling, and cleanup on failure
grep -r 'batch.*process\|progress.*update\|error.*handl\|cleanup.*fail' --include='*.test.*' --include='*.spec.*' | head -20

# Identify streaming and hash computation patterns
grep -r 'transform.*stream\|hash.*stream\|createHash' --include='*.js' --include='*.ts' | head -15

# Verify multipart upload integration
grep -r 'multipart\|upload.*stream' --include='*.js' --include='*.ts' | head -10
```

**Accept when:**
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly
- Transform streams are used for hash computation with proper piping to avoid multiple data passes
- Batch processing partitions documents by cumulative size with meaningful progress increments
- All event emitters and streams have registered error handlers that trigger cleanup operations
- Progress metadata updates occur at defined checkpoints during archive generation

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review verification that new long-running operations follow asynchronous task patterns with streaming interfaces is mandatory. Automated test coverage requirements for asynchronous workflows including error handling and cleanup paths must be confirmed. Architecture review for operations exceeding expected synchronous execution time must verify task orchestration adoption. Pull requests implementing long-running operations as synchronous endpoints MUST be rejected with guidance to adopt task orchestration patterns. Operations that buffer entire archives in memory rather than using streaming interfaces MUST be flagged in code review for refactoring. Missing error handlers on asynchronous operations MUST be identified and required to be added before merge.
</enforcement>