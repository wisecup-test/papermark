# Adopt Stream-Based Middleware for Archive Generation with Progress Tracking: Structured Logging Record

These rules are ALWAYS ACTIVE for archive generation tasks that process multiple documents into compressed formats, long-running operations requiring real-time progress feedback, streaming pipelines with integrity verification, multipart upload workflows, and serverless task execution contexts with event-driven orchestration.

### Rules

- **R-STREAM-001** SHOULD: Structured logging SHOULD record pipeline milestones with contextual metadata including batch numbers, file counts, and size metrics for observability.
- **R-STREAM-002** MUST: Stream middleware MUST be implemented as transform streams with chunk callbacks that update hasher state and invoke progress callbacks without blocking data flow.
- **R-STREAM-003** MUST: Event handlers for error conditions MUST be registered before starting stream processing and MUST coordinate cleanup across all active resources including aborting multipart uploads and destroying stream instances.
- **R-STREAM-004** SHOULD: Progress tracking SHOULD use fractional values representing pipeline stages and batch progress, with metadata updates occurring at milestone boundaries rather than per-chunk to minimize cache layer load.
- **R-STREAM-005** MUST: All stream sources and archive instances MUST have registered error handlers to prevent resource leaks and ensure proper failure propagation.
- **R-STREAM-006** MUST: Stream middleware implementations MUST implement proper backpressure handling to prevent memory accumulation when downstream consumers cannot keep pace with archive generation.
- **R-STREAM-007** SHOULD: Progress metadata updates SHOULD implement retry logic with exponential backoff and log update failures for monitoring to handle cache layer unavailability gracefully.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify stream-based archive generation with progress tracking and error handling.
echo "Running integration tests for stream-based archive generation..."
# [Project-specific test command to be derived from repository]

# Locate the project's linting and static analysis configuration,
# then execute checks to verify stream middleware implements proper backpressure handling.
echo "Running static analysis for backpressure and error propagation..."
# [Project-specific linting command to be derived from repository]

# Identify the project's observability tooling and verify structured logging
# captures archive generation milestones with required contextual metadata.
echo "Verifying structured logging captures required milestone metadata..."
# [Project-specific observability verification to be derived from repository]
```

**Accept when:**
- Integration tests demonstrate successful archive generation with concurrent hashing, progress updates, and multipart upload coordination for datasets exceeding memory limits.
- Error injection tests verify that stream error handlers properly abort uploads, destroy resources, and propagate failures without resource leaks.
- Progress tracking tests confirm metadata updates occur at expected milestones and structured logs contain batch numbers, file counts, and size metrics for all archive operations.
- Static analysis confirms all stream sources and archive instances have registered error handlers.
- Memory usage monitoring demonstrates bounded memory consumption during large dataset processing.

<enforcement>
Claude Code MUST NOT skip or defer verification. Archive generation implementations that do not use stream-based processing must be refactored before deployment. Missing error handlers on stream sources trigger build failures. Progress tracking implementations that do not update metadata at required milestones must be corrected. Structured logging that omits required contextual metadata must be enhanced.
</enforcement>