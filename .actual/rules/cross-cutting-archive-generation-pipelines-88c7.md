# Adopt Stream-Based Middleware for Archive Generation with Progress Tracking: Archive Generation Pipelines

These rules are ALWAYS ACTIVE for archive generation tasks that process multiple documents into compressed formats, long-running operations requiring real-time progress feedback, streaming pipelines with integrity verification, multipart upload workflows, and serverless task execution contexts with event-driven orchestration.

### Rules

- **R-ARCHIVE-001** MUST: Archive generation pipelines MUST use stream-based middleware to intercept data chunks for cross-cutting concerns such as hashing, progress tracking, and error handling.
- **R-ARCHIVE-002** MUST: Stream middleware MUST be implemented as transform streams with chunk callbacks that update hasher state and invoke progress callbacks without blocking data flow.
- **R-ARCHIVE-003** MUST: Event handlers for error conditions MUST be registered before starting stream processing and MUST coordinate cleanup across all active resources including aborting multipart uploads and destroying stream instances.
- **R-ARCHIVE-004** MUST: All stream sources and archive instances MUST have registered error handlers to prevent resource leaks and ensure proper failure propagation.
- **R-ARCHIVE-005** MUST: Progress tracking MUST use fractional values representing pipeline stages and batch progress, with metadata updates occurring at milestone boundaries rather than per-chunk.
- **R-ARCHIVE-006** MUST: Structured logging MUST include correlation identifiers, batch numbers, file counts, and size metrics to enable tracing and debugging across distributed system boundaries.
- **R-ARCHIVE-007** SHOULD: Implement proper backpressure handling in transform streams and monitor memory usage metrics during archive generation.
- **R-ARCHIVE-008** SHOULD: Configure appropriate buffer limits and chunk sizes to prevent memory accumulation if downstream consumers cannot keep pace with archive generation.
- **R-ARCHIVE-009** SHOULD: Implement retry logic for metadata updates with exponential backoff and log update failures for monitoring.
- **R-ARCHIVE-010** SHOULD: Ensure archive generation continues even if progress tracking fails to maintain resilience.

### Verify

```bash
# Discover and run integration tests for stream-based archive generation
find . -type f -name '*test*' -o -name '*spec*' | grep -i archive | head -5
# Execute integration tests
npm test -- --testPathPattern=archive 2>/dev/null || yarn test --testPathPattern=archive 2>/dev/null || pytest -k archive 2>/dev/null

# Locate and execute linting configuration
if [ -f .eslintrc.json ] || [ -f .eslintrc.js ]; then npx eslint 'src/**/*.js' --format json 2>/dev/null | grep -i stream; fi

# Verify structured logging captures milestone metadata
grep -r "correlation\|batch\|file.*count\|size.*metric" src/ --include="*.js" --include="*.ts" | head -10

# Check for error handler registration on stream sources
grep -r "on('error'\|.on(\"error" src/ --include="*.js" --include="*.ts" | wc -l

# Identify observability tooling and verify logging
find . -type f \( -name 'logger*' -o -name '*log*' \) -path '*/src/*' | head -5
```

**Accept when:**
- Integration tests demonstrate successful archive generation with concurrent hashing, progress updates, and multipart upload coordination for datasets exceeding memory limits.
- Error injection tests verify that stream error handlers properly abort uploads, destroy resources, and propagate failures without resource leaks.
- Progress tracking tests confirm metadata updates occur at expected milestones and structured logs contain batch numbers, file counts, and size metrics for all archive operations.
- Code review verification confirms stream middleware implements proper transform stream interfaces with chunk callbacks.
- Static analysis checks confirm all stream sources and archive instances have registered error handlers.
- Observability monitoring confirms structured logging captures required milestone metadata.
- Memory usage remains bounded during archive generation with large datasets.

<enforcement>
Claude Code MUST NOT skip or defer verification. Archive generation implementations that do not use stream-based processing must be refactored before deployment. Missing error handlers on stream sources trigger build failures. Progress tracking implementations that do not update metadata at required milestones must be corrected. Structured logging that omits required contextual metadata must be enhanced.
</enforcement>