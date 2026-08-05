# Adopt Event-Driven Boundaries for Asynchronous Archive Processing: Progress Tracking Update

These rules are ALWAYS ACTIVE for all asynchronous archive generation workflows involving streaming compression, cloud uploads, progress tracking, and long-running document processing tasks that require event-driven coordination between pipeline stages.

### Rules

- **R-EVTDRV-001** SHOULD: Progress tracking SHOULD update metadata state through cache layer operations decoupled from the main processing pipeline.
- **R-EVTDRV-002** MUST: Register error handlers before piping streams together to ensure errors are caught during the pipe setup phase, and always implement cleanup logic in error handlers to prevent resource leaks.
- **R-EVTDRV-003** MUST: Wrap cloud service client send calls in try-catch blocks and implement exponential backoff retry logic for transient failures, while distinguishing between retryable and terminal errors based on service-specific error codes.
- **R-EVTDRV-004** MUST: Structure progress metadata updates to be independent of pipeline success or failure, ensuring that partial progress is recorded even when operations are aborted, to support resume functionality and debugging.
- **R-EVTDRV-005** MUST: Register error, entry, and completion event handlers on all stream objects before initiating stream operations.
- **R-EVTDRV-006** MUST: Implement comprehensive error event handlers on all stream objects, use promise catch chains consistently, and configure runtime to fail fast on unhandled rejections during development.
- **R-EVTDRV-007** MUST: Use once() for single-fire events where appropriate, ensure removeListener calls in cleanup paths, and implement periodic memory profiling in staging environments to prevent event handler memory leaks.
- **R-EVTDRV-008** MUST: Design event handlers to be idempotent where possible, use atomic cache operations for progress updates, and implement optimistic locking for database state transitions to prevent race conditions.

### Verify

```bash
# Discover the project's test execution configuration and run the test suite covering archive generation workflows
# to verify event handler registration and error propagation
echo "Running archive generation workflow tests..."
# (Execute project's test suite for archive processing)

# Locate the project's static analysis or linting configuration and execute checks that detect unhandled promise
# rejections and missing error event handlers
echo "Running static analysis for unhandled promises and missing error handlers..."
# (Execute project's linter/static analysis)

# Identify the project's integration test suite and execute tests that simulate stream errors to verify cleanup
# operations including upload abortion and resource destruction
echo "Running integration tests for stream error scenarios..."
# (Execute project's integration test suite)
```

**Accept when:**
- All archive processing pipelines register error, entry, and completion event handlers before initiating stream operations, verified by code inspection or static analysis.
- Error injection tests demonstrate that failures in any pipeline stage trigger coordinated cleanup including upload abortion and stream destruction without resource leaks.
- Integration tests confirm that progress metadata updates occur independently of pipeline success and that final state correctly reflects completion or failure status.
- Static analysis confirms no unhandled promise rejections or missing error event handlers on stream objects.
- Code review verification shows event handler registration patterns and error propagation logic in all stream-based processing code.

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules in this file are mandatory for archive processing pipelines. Pull requests introducing stream-based processing without complete error event handlers are blocked. Static analysis violations trigger build failures. Production monitoring alerts on unhandled rejection events trigger incident response.
</enforcement>