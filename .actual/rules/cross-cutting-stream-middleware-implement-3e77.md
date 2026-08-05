# Adopt Stream-Based Middleware for Archive Generation with Progress Tracking: Stream Middleware Implement

These rules are ALWAYS ACTIVE for all archive generation tasks that process multiple documents into compressed formats, long-running operations requiring real-time progress feedback, streaming pipelines with integrity verification, multipart upload workflows, and serverless task execution contexts with event-driven orchestration.

### Rules

- **R-STREAM-001** MUST: Stream middleware MUST implement the transform stream interface with chunk processing callbacks that preserve data integrity while performing side effects.
- **R-STREAM-002** MUST: Event handlers for error conditions MUST be registered before starting stream processing and MUST coordinate cleanup across all active resources including aborting multipart uploads and destroying stream instances.
- **R-STREAM-003** MUST: All stream sources and archive instances MUST have registered error handlers to prevent resource leaks and ensure proper failure propagation.
- **R-STREAM-004** MUST: Progress tracking MUST update metadata at specific milestone boundaries (0.05, 0.1, batch-specific values) rather than per-chunk to minimize cache layer load.
- **R-STREAM-005** MUST: Structured logging MUST include correlation identifiers, batch numbers, file counts, and size metrics to enable tracing and debugging of long-running archive generation operations.
- **R-STREAM-006** SHOULD: Implement proper backpressure handling in transform streams and monitor memory usage metrics during archive generation with appropriate buffer limits and chunk sizes.
- **R-STREAM-007** SHOULD: Implement retry logic for metadata updates with exponential backoff and log update failures for monitoring to handle cache layer unavailability gracefully.

### Verify

```bash
# Discover and run integration tests for stream-based archive generation
find . -type f -name '*test*' -o -name '*spec*' | grep -E '(archive|stream)' | head -5
# Execute integration tests
npm test -- --testPathPattern='archive|stream' 2>&1 || yarn test --testPathPattern='archive|stream' 2>&1 || pytest -k 'archive or stream' 2>&1

# Locate and execute linting/static analysis
if [ -f '.eslintrc' ] || [ -f '.eslintrc.json' ]; then npm run lint 2>&1 || yarn lint 2>&1; fi
if [ -f 'pyproject.toml' ] || [ -f 'setup.py' ]; then pylint . 2>&1 || flake8 . 2>&1; fi

# Verify structured logging captures archive generation milestones
grep -r 'correlation\|batch\|fileCount\|size' --include='*.js' --include='*.ts' --include='*.py' . 2>/dev/null | head -10

# Verify error handlers are registered on stream sources
grep -r '\.on.*error' --include='*.js' --include='*.ts' . 2>/dev/null | grep -E '(archive|stream)' | head -10
```

**Accept when:**
- Integration tests demonstrate successful archive generation with concurrent hashing, progress updates, and multipart upload coordination for datasets exceeding memory limits.
- Error injection tests verify that stream error handlers properly abort uploads, destroy resources, and propagate failures without resource leaks.
- Progress tracking tests confirm metadata updates occur at expected milestones and structured logs contain batch numbers, file counts, and size metrics for all archive operations.
- Static analysis checks confirm all stream sources and archive instances have registered error handlers.
- Memory usage remains bounded during archive generation with large datasets as verified by integration tests.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for archive generation implementations. Violations must be resolved before merge. Exception process requires explicit approval from the architecture review board for small archives under defined size thresholds, temporary progress tracking degradation during cache layer outages, or alternative hashing strategies for specific compliance requirements.
</enforcement>