# Adopt Stream-Based Middleware for Archive Generation with Progress Tracking: Implementations That Depend

These rules are ALWAYS ACTIVE for archive generation tasks that process multiple documents into compressed formats, long-running operations requiring real-time progress feedback, streaming pipelines with integrity verification, multipart upload workflows, and serverless task execution contexts with event-driven orchestration.

### Rules

- **R-STREAM-001** MUST: Implementations that depend on versioned cloud SDK clients, streaming libraries, or task orchestration frameworks MUST discover the exact resolved version from the project's dependency lock artifact before using any API.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify stream-based archive generation with progress tracking and error handling.
echo "Running integration tests for stream-based archive generation..."

# Locate the project's linting and static analysis configuration, then execute checks
# to verify stream middleware implements proper backpressure handling and error propagation.
echo "Verifying stream middleware backpressure and error propagation..."

# Identify the project's observability tooling and verify that structured logging
# captures archive generation milestones with required contextual metadata.
echo "Verifying structured logging captures required milestone metadata..."
```

**Accept when:**
- Integration tests demonstrate successful archive generation with concurrent hashing, progress updates, and multipart upload coordination for datasets exceeding memory limits.
- Error injection tests verify that stream error handlers properly abort uploads, destroy resources, and propagate failures without resource leaks.
- Progress tracking tests confirm metadata updates occur at expected milestones and structured logs contain batch numbers, file counts, and size metrics for all archive operations.
- Code review verification confirms stream middleware implements proper transform stream interfaces with chunk callbacks.
- Static analysis checks confirm all stream sources and archive instances have registered error handlers.
- Observability monitoring confirms structured logging captures required milestone metadata.

<enforcement>
Claude Code MUST NOT skip or defer verification. Archive generation implementations that do not use stream-based processing must be refactored before deployment. Missing error handlers on stream sources trigger build failures. Progress tracking implementations that do not update metadata at required milestones must be corrected. Structured logging that omits required contextual metadata must be enhanced.
</enforcement>