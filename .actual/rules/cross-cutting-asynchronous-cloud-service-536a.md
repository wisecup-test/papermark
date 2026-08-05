# Adopt Event-Driven Boundaries for Asynchronous Archive Processing: Asynchronous Cloud Service

These rules are ALWAYS ACTIVE for all asynchronous archive generation workflows involving streaming compression, cryptographic hashing, multipart cloud uploads, database state updates, and serverless function invocations that require event-driven coordination.

### Rules

- **R-ASYNC-001** MUST: Asynchronous cloud service invocations MUST use event-driven client send operations that return promises for result handling.

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
Clause Code MUST NOT skip or defer verification. All stream-based processing code must be reviewed for complete event handler registration patterns and error propagation logic. Static analysis violations for unhandled promises or missing error handlers trigger build failures. Production monitoring alerts on unhandled rejection events require incident response and root cause analysis.
</enforcement>