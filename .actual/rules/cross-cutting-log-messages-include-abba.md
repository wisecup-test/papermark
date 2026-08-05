# Adopt Structured Logging with Contextual Metadata for Asynchronous Archive Operations: Log Messages Include

These rules are ALWAYS ACTIVE for all asynchronous archive operations, multi-stage document processing workflows, batch processing operations, cloud storage uploads, lambda invocations, and CSV generation and export operations.

### Rules

- **R-LOG-001** MAY: Log messages MAY include conditional text formatting based on operation characteristics such as single versus multi-batch processing.

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

<enforcement>
Code review verification that new logging statements include structured metadata objects with appropriate operational context is mandatory. Static analysis rules that flag logging statements missing structured metadata in designated operation types must pass. Integration test validation that progress metadata updates occur at expected milestones with correct values is required. Claude Code MUST NOT skip or defer verification.
</enforcement>