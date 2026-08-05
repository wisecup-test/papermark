# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Batch Processing Large

These rules are ALWAYS ACTIVE for all archive generation tasks, dataroom freeze operations, and long-running workflows involving document collection, batch processing, streaming compression, hash computation, and multipart uploads to object storage.

### Rules

- **R-ASYNC-001** MUST: Batch processing of large document sets MUST partition work into size-bounded batches with per-batch progress reporting.
- **R-ASYNC-002** MUST: Archive operations MUST be implemented as asynchronous task functions that coordinate multiple workflows including document retrieval, batch processing, streaming compression, hash computation, and multipart upload.
- **R-ASYNC-003** MUST: Use transform streams to compute cryptographic hashes incrementally during archive creation and upload, piping archive output through hash computation transforms before passing to upload streams.
- **R-ASYNC-004** MUST: Structure batch processing to partition documents by cumulative size rather than count alone, ensuring batches remain within memory and processing time constraints.
- **R-ASYNC-005** MUST: Register error handlers on all event emitters and streams to propagate failures through the workflow, ensuring cleanup operations such as upload abortion execute before task failure.
- **R-ASYNC-006** SHOULD: Provide incremental progress feedback during long-running operations through metadata checkpoint updates at meaningful intervals.
- **R-ASYNC-007** SHOULD: Implement retry logic with exponential backoff for task orchestration infrastructure failures.
- **R-ASYNC-008** MAY: Synchronous implementations may be used for operations with proven completion times under 10 seconds with documented justification and technical lead approval.

### Verify

```bash
# Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
find . -type f -name '*.json' -o -name '*.yaml' -o -name '*.yml' | xargs grep -l 'task\|orchestration\|archive' | head -5

# Locate the test suite covering archive task execution
find . -type f \( -name '*test*.js' -o -name '*test*.ts' -o -name '*spec*.js' -o -name '*spec*.ts' \) | xargs grep -l 'archive\|batch.*process' | head -5

# Verify tests exist for batch processing, progress updates, error handling, and cleanup
grep -r 'batch.*process\|progress.*update\|error.*handl\|cleanup' --include='*test*' --include='*spec*' . | wc -l

# Identify integration test infrastructure
find . -type f -path '*/test*' -o -path '*/integration*' | grep -E '\.(js|ts)$' | head -5

# Verify stream usage for hash computation
grep -r 'transform.*stream\|hash.*stream\|createHash' --include='*.js' --include='*.ts' . | grep -v node_modules | head -10
```

**Accept when:**
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly
- All asynchronous operations have registered error handlers verified through code review
- Batch processing partitions by cumulative size with documented constraints
- Transform streams are used for incremental hash computation without buffering entire archives in memory

<enforcement>
Clause Code MUST NOT skip or defer verification. All archive operations exceeding 10 seconds expected execution time MUST follow asynchronous task patterns with streaming interfaces. Operations buffering entire archives in memory or lacking error handlers on streams are violations requiring remediation before merge.
</enforcement>