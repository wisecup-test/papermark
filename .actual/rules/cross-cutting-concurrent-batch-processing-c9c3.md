# Adopt Event-Driven Boundaries for Asynchronous Archive Processing: Concurrent Batch Processing

These rules are ALWAYS ACTIVE for all asynchronous archive generation workflows, streaming compression pipelines, cloud storage upload operations, and long-running document processing tasks that require progress tracking and coordinated error handling across multiple concurrent streams.

### Rules

- **R-EVDRIVE-001** MAY: Concurrent batch processing operations MAY execute in parallel when batch independence is guaranteed by the data access pattern.
- **R-EVDRIVE-002** MUST: Register error, entry, and completion event handlers on all stream objects before initiating pipe operations to ensure errors are caught during setup phase.
- **R-EVDRIVE-003** MUST: Implement cleanup logic in all error event handlers to prevent resource leaks, including upload abortion and stream destruction.
- **R-EVDRIVE-004** MUST: Wrap cloud service client send calls in try-catch blocks and implement exponential backoff retry logic for transient failures.
- **R-EVDRIVE-005** MUST: Distinguish between retryable and terminal errors based on service-specific error codes in cloud service operations.
- **R-EVDRIVE-006** MUST: Structure progress metadata updates to be independent of pipeline success or failure to support resume functionality and debugging.
- **R-EVDRIVE-007** SHOULD: Use once() for single-fire events where appropriate and ensure removeListener calls in cleanup paths to prevent memory leaks.
- **R-EVDRIVE-008** SHOULD: Design event handlers to be idempotent where possible and use atomic cache operations for progress updates.
- **R-EVDRIVE-009** SHOULD: Implement optimistic locking for database state transitions in concurrent event handler scenarios.
- **R-EVDRIVE-010** MUST: Configure runtime to fail fast on unhandled promise rejections during development.

### Verify

```bash
# Discover the project's test execution configuration and run the test suite covering archive generation workflows
# to verify event handler registration and error propagation
echo "Running archive generation workflow tests..."
# [Execute project test suite for archive processing]

# Locate the project's static analysis or linting configuration and execute checks that detect unhandled promise
# rejections and missing error event handlers
echo "Running static analysis for unhandled promises and missing error handlers..."
# [Execute linting/static analysis configured in project]

# Identify the project's integration test suite and execute tests that simulate stream errors to verify cleanup
# operations including upload abortion and resource destruction
echo "Running integration tests for stream error scenarios..."
# [Execute integration test suite with error injection]
```

**Accept when:**
- All archive processing pipelines register error, entry, and completion event handlers before initiating stream operations, verified by code inspection or static analysis.
- Error injection tests demonstrate that failures in any pipeline stage trigger coordinated cleanup including upload abortion and stream destruction without resource leaks.
- Integration tests confirm that progress metadata updates occur independently of pipeline success and that final state correctly reflects completion or failure status.
- Static analysis confirms no unhandled promise rejections or missing error event handlers on stream objects.
- Cloud service operations include exponential backoff retry logic with proper error code classification.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for archive processing code. Code review MUST verify event handler registration patterns and error propagation logic. Static analysis violations for unhandled promises or missing error handlers MUST trigger build failures. Pull requests introducing stream-based processing without complete error event handlers MUST be blocked until handlers are added and tested.
</enforcement>