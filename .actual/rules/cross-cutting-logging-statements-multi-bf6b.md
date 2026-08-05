# Adopt Structured Logging with Contextual Metadata for Asynchronous Archive Operations: Logging Statements Multi

These rules are ALWAYS ACTIVE for all logging statements in multi-stage asynchronous archive operations, batch processing workflows, cloud storage uploads, lambda invocations, and CSV generation operations.

### Rules

- **R-LOG-001** MUST: All logging statements for multi-stage asynchronous operations MUST include structured metadata objects containing relevant operational context such as identifiers, counts, sizes, and batch information.

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
Verification is mandatory. Code review MUST confirm structured metadata presence. Static analysis MUST flag missing metadata. Integration tests MUST validate progress updates at milestones. Violations block CI pipeline until corrected.
</enforcement>