# Adopt Stream-Based Middleware for Archive Generation with Progress Tracking: Archive Operations Batch

These rules are ALWAYS ACTIVE for all archive generation tasks that process multiple documents into compressed formats, long-running operations requiring real-time progress feedback, streaming pipelines with integrity verification, multipart upload workflows, and serverless task execution contexts with event-driven orchestration.

### Rules

- **R-ARCHIVE-001** SHOULD: Archive operations SHOULD batch documents based on size constraints and generate separate archives with coordinated progress tracking across batches.
- **R-ARCHIVE-002** MUST: Stream middleware MUST implement proper transform stream interfaces with chunk callbacks that update hasher state and invoke progress callbacks without blocking data flow.
- **R-ARCHIVE-003** MUST: All stream sources and archive instances MUST have registered error handlers before starting stream processing to coordinate cleanup across active resources.
- **R-ARCHIVE-004** MUST: Error handlers MUST be registered before stream processing begins and coordinate cleanup across all active resources including aborting multipart uploads and destroying stream instances.
- **R-ARCHIVE-005** SHOULD: Progress tracking SHOULD use fractional values representing pipeline stages and batch progress, with metadata updates occurring at milestone boundaries rather than per-chunk.
- **R-ARCHIVE-006** MUST: Structured logging MUST include correlation identifiers, batch numbers, file counts, and size metrics to enable tracing across distributed system boundaries.
- **R-ARCHIVE-007** MUST: Stream backpressure handling MUST be implemented in transform streams with appropriate buffer limits and chunk sizes to prevent memory accumulation.
- **R-ARCHIVE-008** SHOULD: Progress metadata updates SHOULD implement retry logic with exponential backoff and log failures for monitoring.
- **R-ARCHIVE-009** MAY: Small archive operations under defined size thresholds may use in-memory generation with approval from the architecture review board.

### Verify

```bash
# Discover and run integration tests for stream-based archive generation
find . -type f -name '*test*' -o -name '*spec*' | grep -i archive | head -5
# Execute integration tests
npm test -- --testPathPattern=archive 2>&1 || yarn test --testPathPattern=archive 2>&1 || pytest -k archive 2>&1

# Locate and execute linting configuration
if [ -f .eslintrc.json ] || [ -f .eslintrc.js ]; then npx eslint 'src/**/*.{js,ts}' --format json 2>&1; fi
if [ -f pyproject.toml ] || [ -f setup.py ]; then pylint src/ 2>&1; fi

# Verify structured logging captures archive generation milestones
grep -r "correlation\|batch\|file.*count\|size.*metric" src/ --include="*.js" --include="*.ts" --include="*.py" 2>&1 | head -20

# Check for stream error handler registration
grep -r "on.*error\|\.catch\|try.*catch" src/ --include="*.js" --include="*.ts" --include="*.py" | grep -i "stream\|archive" 2>&1 | head -20
```

**Accept when:**
- Integration tests demonstrate successful archive generation with concurrent hashing, progress updates, and multipart upload coordination for datasets exceeding memory limits.
- Error injection tests verify that stream error handlers properly abort uploads, destroy resources, and propagate failures without resource leaks.
- Progress tracking tests confirm metadata updates occur at expected milestones and structured logs contain batch numbers, file counts, and size metrics for all archive operations.
- Static analysis confirms all stream sources and archive instances have registered error handlers before stream processing begins.
- Memory usage monitoring shows bounded memory consumption during archive generation of large datasets.
- Structured logging output includes correlation identifiers, batch numbers, file counts, and size metrics for tracing.

<enforcement>
Claude Code MUST NOT skip or defer verification. Archive generation implementations that do not use stream-based processing must be refactored before deployment. Missing error handlers on stream sources trigger build failures. Progress tracking implementations that do not update metadata at required milestones must be corrected. Structured logging that omits required contextual metadata must be enhanced.
</enforcement>