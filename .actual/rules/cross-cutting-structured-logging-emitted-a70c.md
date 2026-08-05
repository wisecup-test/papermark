# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Structured Logging Emitted

These rules are ALWAYS ACTIVE for all long-running archive operations that process multiple documents, generate compressed artifacts, perform multipart uploads to object storage, or coordinate multiple asynchronous data retrieval and transformation steps.

### Rules

- **R-ASYNC-LOG-001** SHOULD: Structured logging SHOULD be emitted at workflow boundaries including operation start, batch completion, and final artifact generation with relevant context metadata.
- **R-ASYNC-LOG-002** MUST: All asynchronous operations MUST register error handlers that propagate failures through the workflow and trigger cleanup operations such as upload abortion before task failure.
- **R-ASYNC-LOG-003** MUST: Archive operations MUST use transform streams to compute cryptographic hashes incrementally during archive creation and upload, piping archive output through hash computation transforms before passing to upload streams.
- **R-ASYNC-LOG-004** MUST: Batch processing MUST partition documents by cumulative size rather than count alone to ensure batches remain within memory and processing time constraints.
- **R-ASYNC-LOG-005** SHOULD: Progress metadata updates SHOULD be emitted at meaningful checkpoints during long-running operations to provide incremental user feedback on archive creation status.
- **R-ASYNC-LOG-006** MUST: Archive operations MUST NOT buffer entire archives in memory; streaming interfaces MUST be used for compression and upload operations.

### Verify

```bash
# Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
find . -type f -name '*.json' -o -name '*.yaml' -o -name '*.yml' | xargs grep -l 'task\|orchestration' | head -5

# Locate the test suite covering archive task execution
find . -type f \( -name '*test*' -o -name '*spec*' \) | grep -i archive | head -10

# Verify tests exist for batch processing, progress updates, error handling, and cleanup
grep -r 'batch\|progress\|error.*handler\|cleanup' --include='*test*' --include='*spec*' | grep -i archive | head -20

# Identify streaming and hash computation patterns
grep -r 'transform.*stream\|hash.*stream\|createHash' --include='*.js' --include='*.ts' | head -10

# Verify multipart upload error handling
grep -r 'abort.*upload\|cleanup.*multipart' --include='*.js' --include='*.ts' | head -10
```

**Accept when:**
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly
- Structured logging is present at operation start, batch completion, and final artifact generation with relevant context metadata
- All asynchronous operations have registered error handlers that execute cleanup before task failure
- Transform streams are used for hash computation with no buffering of entire archives in memory
- Batch processing partitions documents by cumulative size with meaningful progress increments

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review approval of long-running archive operations.
</enforcement>