# Adopt Event-Driven Boundaries for Asynchronous Archive Processing: Error Events Any

These rules are ALWAYS ACTIVE for all asynchronous archive generation workflows involving streaming compression, cryptographic hashing, multipart cloud uploads, and database state updates that require event-driven coordination between pipeline stages.

### Rules

- **R-EVTDRV-001** MUST: Error events from any stream in the pipeline MUST propagate to all dependent streams and trigger cleanup operations including upload abortion and resource destruction.

### Verify

```bash
# Discover the project's test execution configuration and run the test suite covering archive generation workflows to verify event handler registration and error propagation
# Locate the project's static analysis or linting configuration and execute checks that detect unhandled promise rejections and missing error event handlers
# Identify the project's integration test suite and execute tests that simulate stream errors to verify cleanup operations including upload abortion and resource destruction
```

**Accept when:**
- All archive processing pipelines register error, entry, and completion event handlers before initiating stream operations, verified by code inspection or static analysis
- Error injection tests demonstrate that failures in any pipeline stage trigger coordinated cleanup including upload abortion and stream destruction without resource leaks
- Integration tests confirm that progress metadata updates occur independently of pipeline success and that final state correctly reflects completion or failure status

<enforcement>
Clause Code MUST NOT skip or defer verification. All stream-based processing code must include complete error event handlers and pass error injection tests before merge.
</enforcement>