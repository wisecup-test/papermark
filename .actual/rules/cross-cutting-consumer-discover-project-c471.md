# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Consumer Discover Project

These rules are ALWAYS ACTIVE for all long-running archive operations, dataroom freeze workflows, batch processing tasks, streaming compression, multipart uploads, and any operation coordinating multiple asynchronous workflows that exceed typical HTTP request timeouts.

### Rules

- **R-ASYNC-001** MUST: Discover the project's dependency lock artifact and resolve the exact locked versions of all asynchronous runtime libraries, SDK clients, and streaming utilities before implementation.
- **R-ASYNC-002** MUST: Implement archive operations as task functions that accept payload objects containing dataroom identifiers and team context, coordinating multiple asynchronous workflows including document retrieval, batch processing, streaming compression, hash computation, and multipart upload.
- **R-ASYNC-003** MUST: Use transform streams to compute cryptographic hashes incrementally during archive creation and upload, piping archive output through hash computation transforms before passing to upload streams to avoid multiple passes over data.
- **R-ASYNC-004** MUST: Register error handlers on all event emitters and streams to propagate failures through the workflow, ensuring cleanup operations such as upload abortion execute before task failure.
- **R-ASYNC-005** SHOULD: Structure batch processing to partition documents by cumulative size rather than count alone, ensuring batches remain within memory and processing time constraints while providing meaningful progress increments.
- **R-ASYNC-006** SHOULD: Implement retry logic with exponential backoff for task orchestration infrastructure failures, monitor task execution metrics, and provide manual retry mechanisms for failed operations.
- **R-ASYNC-007** SHOULD: Treat metadata update failures as non-fatal warnings, implement timeout-based progress estimation as fallback, and ensure final operation status is always persisted.
- **R-ASYNC-008** MAY: Use synchronous implementations for operations with proven completion times under 10 seconds with documented justification and technical lead approval.

### Verify

```bash
# Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
grep -r "task.*archive\|freeze.*task" . --include="*.json" --include="*.yaml" --include="*.yml" --include="*.ts" --include="*.js"

# Locate the test suite covering archive task execution
find . -path "*/test*" -name "*archive*" -o -path "*/test*" -name "*freeze*" | head -20

# Verify tests exist for batch processing, progress updates, error handling, and cleanup on failure
grep -r "batch.*process\|progress.*update\|error.*handler\|cleanup" . --include="*.test.ts" --include="*.spec.ts" --include="*.test.js" --include="*.spec.js"

# Identify the project's integration test infrastructure
find . -path "*/integration*" -type f -name "*.ts" -o -name "*.js" | head -10

# Verify transform streams are used for hash computation
grep -r "transform.*stream\|hash.*stream\|createHash" . --include="*.ts" --include="*.js" | grep -v node_modules

# Verify error handlers are registered on streams and event emitters
grep -r "\.on.*error\|catch.*error\|error.*handler" . --include="*.ts" --include="*.js" | grep -v node_modules
```

**Accept when:**
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly
- Transform streams are used for incremental hash computation during archive creation and upload
- All event emitters and streams have registered error handlers that propagate failures and trigger cleanup
- Batch processing partitions documents by cumulative size with meaningful progress increments
- Dependency lock artifact is consulted and exact versions of asynchronous libraries are documented

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory and must be verified before code is committed. Code review MUST reject implementations that buffer entire archives in memory, use synchronous endpoints for operations exceeding 10 seconds, or lack error handlers on asynchronous operations. Static analysis MUST flag missing error handlers. Exception requests require performance testing data and technical lead approval.
</enforcement>