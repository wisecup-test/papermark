# Adopt Structured Logging with Contextual Metadata for Asynchronous Archive Operations: Error Logging Stream

These rules are ALWAYS ACTIVE for all asynchronous archive operations, multi-stage document processing workflows, batch processing operations, cloud storage uploads, lambda invocations, and CSV generation and export operations.

### Rules

- **R-LOGGING-001** SHOULD: Error logging in stream-based operations SHOULD capture error context through structured metadata and trigger cleanup operations for partial uploads.

### Verify

```bash
# Discover and execute the project's static analysis configuration to verify that logging statements in multi-stage operations include structured metadata objects.
# Discover and run the project's test suite to verify that progress metadata updates occur at expected operation milestones and include both numeric progress and status text.
# Discover and execute the project's log validation tooling to verify that batch processing operations emit batch-level metadata including sequence numbers, counts, and size metrics.
```

**Accept when:**
- All logging statements in multi-stage asynchronous operations include structured metadata objects with relevant operational context.
- Progress tracking mechanism updates both numeric progress indicators and status text at key operation milestones.
- Batch processing operations emit structured metadata including batch number, total batches, file counts, and size metrics at batch start and completion.
- Error handlers in stream-based operations log structured error context and trigger cleanup operations such as aborting partial uploads.

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review verification, static analysis rules, and integration test validation are mandatory before merge approval.
</enforcement>