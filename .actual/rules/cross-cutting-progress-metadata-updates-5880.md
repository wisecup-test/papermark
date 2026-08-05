# Adopt Stream-Based Middleware for Archive Generation with Progress Tracking: Progress Metadata Updates

These rules are ALWAYS ACTIVE for all archive generation tasks that process multiple documents into compressed formats, long-running operations requiring real-time progress feedback, streaming pipelines with integrity verification, multipart upload workflows, and serverless task execution contexts with event-driven orchestration.

### Rules

- **R-STREAM-001** MUST: Progress metadata updates MUST be coordinated through a cache layer that supports atomic set operations for tracking pipeline milestones.
- **R-STREAM-002** MUST: Stream middleware MUST be implemented as transform streams with chunk callbacks that update hasher state and invoke progress callbacks without blocking data flow.
- **R-STREAM-003** MUST: Event handlers for error conditions MUST be registered before starting stream processing and MUST coordinate cleanup across all active resources including aborting multipart uploads and destroying stream instances.
- **R-STREAM-004** MUST: All stream sources and archive instances MUST have registered error handlers to prevent resource leaks and ensure proper failure propagation.
- **R-STREAM-005** MUST: Structured logging MUST include correlation identifiers, batch numbers, file counts, and size metrics to enable tracing of archive generation operations across distributed system boundaries.
- **R-STREAM-006** SHOULD: Progress tracking SHOULD use fractional values representing pipeline stages and batch progress, with metadata updates occurring at milestone boundaries rather than per-chunk to minimize cache layer load.
- **R-STREAM-007** SHOULD: Stream backpressure handling SHOULD be implemented in transform streams with appropriate buffer limits and chunk sizes, and memory usage metrics SHOULD be monitored during archive generation.
- **R-STREAM-008** SHOULD: Retry logic for metadata updates SHOULD implement exponential backoff, and update failures SHOULD be logged for monitoring while archive generation continues.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify stream-based archive generation with progress tracking and error handling
find . -name 'package.json' -o -name 'pyproject.toml' -o -name 'go.mod' -o -name 'Cargo.toml' | head -1

# Locate the project's linting and static analysis configuration
find . -name '.eslintrc*' -o -name 'pylintrc' -o -name '.golangci.yml' -o -name 'clippy.toml' | head -1

# Identify the project's observability tooling configuration
find . -name 'winston.config.*' -o -name 'pino.config.*' -o -name 'logging.conf' -o -name 'otel.config.*' | head -1

# Verify stream middleware implements proper transform stream interfaces
grep -r "Transform\|transform\|pipe\|on('data'\|on('error'" --include='*.js' --include='*.ts' --include='*.py' . | grep -E '(archive|stream)' | head -20

# Verify error handlers are registered on stream sources
grep -r "on('error'\|.catch(\|try.*catch" --include='*.js' --include='*.ts' --include='*.py' . | grep -E '(archive|stream|source)' | head -20

# Verify structured logging captures milestone metadata
grep -r "batch\|milestone\|progress\|correlation" --include='*.js' --include='*.ts' --include='*.py' . | grep -i 'log\|logger' | head -20
```

**Accept when:**
- Integration tests demonstrate successful archive generation with concurrent hashing, progress updates, and multipart upload coordination for datasets exceeding memory limits.
- Error injection tests verify that stream error handlers properly abort uploads, destroy resources, and propagate failures without resource leaks.
- Progress tracking tests confirm metadata updates occur at expected milestones and structured logs contain batch numbers, file counts, and size metrics for all archive operations.
- Code review verification confirms stream middleware implements proper transform stream interfaces with chunk callbacks.
- Static analysis checks confirm all stream sources and archive instances have registered error handlers.
- Observability monitoring confirms structured logging captures required milestone metadata with correlation identifiers.
- Memory usage remains bounded during archive generation with large datasets exceeding available RAM.

<enforcement>
Claude Code MUST NOT skip or defer verification. Archive generation implementations that do not use stream-based processing must be refactored before deployment. Missing error handlers on stream sources trigger build failures. Progress tracking implementations that do not update metadata at required milestones must be corrected. Structured logging that omits required contextual metadata must be enhanced.
</enforcement>