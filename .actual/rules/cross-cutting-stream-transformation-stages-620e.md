# Adopt Event-Driven Boundaries for Asynchronous Archive Processing: Stream Transformation Stages

These rules are ALWAYS ACTIVE for all asynchronous archive generation workflows involving streaming compression, cryptographic hashing, multipart cloud uploads, database state updates, and long-running document processing tasks that require progress tracking and cancellation support.

### Rules

- **R-STREAM-001** MUST: Stream transformation stages that compute derived values MUST use transform streams with chunk-based event handlers to maintain pipeline backpressure.

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
Clause Code MUST NOT skip or defer verification. All stream-based processing code must be reviewed for complete error event handler registration and tested for proper error propagation and resource cleanup before merge.
</enforcement>