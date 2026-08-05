# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Error Handlers Registered

These rules are ALWAYS ACTIVE for all long-running archive operations that involve document collection, batch processing, streaming compression, hash computation, and multipart uploads to object storage, as well as any asynchronous workflows requiring explicit error propagation and resource cleanup.

### Rules

- **R-ASYNC-001** MUST: Error handlers MUST be registered on all asynchronous stream and event sources to propagate failures and trigger cleanup operations including upload abortion.
- **R-ASYNC-002** MUST: Archive operations MUST be implemented as task functions that accept payload objects containing dataroom identifiers and team context, coordinating multiple asynchronous workflows including document retrieval, batch processing, streaming compression, hash computation, and multipart upload.
- **R-ASYNC-003** MUST: Transform streams MUST be used to compute cryptographic hashes incrementally during archive creation and upload, piping archive output through hash computation transforms before passing to upload streams to avoid multiple passes over data.
- **R-ASYNC-004** MUST: Batch processing MUST partition documents by cumulative size rather than count alone, ensuring batches remain within memory and processing time constraints while providing meaningful progress increments.
- **R-ASYNC-005** MUST: Error handlers on all event emitters and streams MUST propagate failures through the workflow, ensuring cleanup operations such as upload abortion execute before task failure.
- **R-ASYNC-006** SHOULD: Progress tracking SHOULD be implemented through incremental metadata updates at checkpoints (0.05, 0.1, per-batch) to provide user feedback on archive creation status.
- **R-ASYNC-007** SHOULD: Retry logic with exponential backoff SHOULD be implemented to handle task orchestration infrastructure failures or quota limits.

### Verify

```bash
# Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
find . -type f -name '*.json' -o -name '*.yaml' -o -name '*.yml' | xargs grep -l 'task\|orchestration' | head -5

# Locate the test suite covering archive task execution
find . -type f \( -name '*test*' -o -name '*spec*' \) | xargs grep -l 'archive\|freeze' | head -10

# Verify tests exist for batch processing, progress updates, error handling, and cleanup on failure
grep -r 'batch\|progress\|error.*handler\|cleanup' --include='*test*' --include='*spec*' | grep -i archive | head -20

# Identify stream and event emitter usage in archive operations
grep -r 'on.*error\|.catch\|addEventListener' --include='*.js' --include='*.ts' | grep -i archive | head -15

# Verify multipart upload error handling
grep -r 'abort\|cleanup' --include='*.js' --include='*.ts' | grep -i 'upload\|multipart' | head -10
```

**Accept when:**
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly
- All asynchronous stream and event sources have registered error handlers that propagate failures
- Batch processing partitions documents by cumulative size with meaningful progress increments
- Transform streams are used for incremental hash computation without buffering entire archives
- Cleanup operations execute on task failure, including multipart upload abortion

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory and violations must be identified through code review, automated test coverage requirements, and architecture review before merge.
</enforcement>