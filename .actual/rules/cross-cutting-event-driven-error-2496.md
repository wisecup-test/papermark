# Adopt Stream-Based Middleware for Archive Generation with Progress Tracking: Event Driven Error

These rules are ALWAYS ACTIVE for all archive generation tasks that process multiple documents into compressed formats, long-running operations requiring real-time progress feedback, streaming pipelines with integrity verification, multipart upload workflows, and serverless task execution contexts with event-driven orchestration.

### Rules

- **R-STREAM-001** MUST: Event-driven error handlers MUST be attached to all stream sources and archive instances to propagate failures and trigger cleanup operations including upload abortion.

### Verify

```bash
# Discover the project's test execution configuration and run integration tests
# that verify stream-based archive generation with progress tracking and error handling.
find . -name 'package.json' -o -name 'pyproject.toml' -o -name 'go.mod' | head -1

# Locate the project's linting and static analysis configuration, then execute checks
# to verify stream middleware implements proper backpressure handling and error propagation.
find . -name '.eslintrc*' -o -name 'pylintrc' -o -name '.golangci.yml' | head -1

# Identify the project's observability tooling and verify that structured logging
# captures archive generation milestones with required contextual metadata.
grep -r "structured.*log\|winston\|pino\|bunyan\|python.*logging" . --include="*.json" --include="*.toml" --include="*.yaml" | head -5
```

**Accept when:**
- Integration tests demonstrate successful archive generation with concurrent hashing, progress updates, and multipart upload coordination for datasets exceeding memory limits.
- Error injection tests verify that stream error handlers properly abort uploads, destroy resources, and propagate failures without resource leaks.
- Progress tracking tests confirm metadata updates occur at expected milestones and structured logs contain batch numbers, file counts, and size metrics for all archive operations.
- Code review verification confirms stream middleware implements proper transform stream interfaces with chunk callbacks.
- Static analysis checks confirm all stream sources and archive instances have registered error handlers.
- Observability monitoring confirms structured logging captures required milestone metadata.

<enforcement>
Claude Code MUST NOT skip or defer verification. All stream sources and archive instances MUST have registered error handlers before deployment. Missing error handlers trigger build failures and must be resolved before merge. Archive generation implementations that do not use stream-based processing must be refactored before production deployment.
</enforcement>