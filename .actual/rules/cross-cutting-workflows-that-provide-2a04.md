# Adopt Modular Core Library Architecture with Observability and Event-Driven Boundaries: Workflows That Provide

These rules are ALWAYS ACTIVE for feature modules within the enterprise edition that coordinate external services, long-running workflows that require operational visibility and progress tracking, data transformation pipelines that process user-generated content or sensitive data, and API task definitions that expose asynchronous operations to orchestration layers.

### Rules

- **R-MODULAR-001** SHOULD: Workflows that provide user-facing feedback SHOULD use the cache layer to persist progress metadata and status text at incremental completion points.
- **R-MODULAR-002** MUST: Feature modules MUST import core infrastructure libraries from the established internal library path, not directly instantiate external service clients.
- **R-MODULAR-003** MUST: Long-running workflows MUST emit structured log events at key lifecycle points with required contextual metadata fields.
- **R-MODULAR-004** MUST: Event-driven stream processing MUST register error handlers at each transformation stage to provide granular failure diagnostics.
- **R-MODULAR-005** SHOULD: Cache metadata schemas for progress tracking SHOULD include both numeric progress indicators and human-readable status text, with idempotent update semantics.
- **R-MODULAR-006** MUST: All versioned library usage MUST be verified against the project's lock artifact before implementation, not training-data recall.

### Verify

```bash
# Discover the project's static analysis configuration and execute linting rules
# that enforce import path conventions for core library usage
find . -name '.eslintrc*' -o -name 'tsconfig.json' -o -name '.pylintrc' | head -1

# Locate the project's test suite directory and run integration tests
# that verify structured logging output contains required contextual metadata fields
find . -type d -name '__tests__' -o -name 'test' -o -name 'tests' | head -1

# Identify the project's dependency verification script and execute it
# to confirm that all versioned libraries match the resolved versions in the lock artifact
find . -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'Pipfile.lock' -o -name 'go.sum' | head -1
```

**Accept when:**
- Static analysis passes with no violations of core library import path conventions, confirming that feature modules do not directly instantiate external service clients.
- Integration tests confirm that all long-running workflows emit structured log events at key lifecycle points with the required contextual metadata fields.
- Dependency verification confirms that the runtime environment matches the exact versions specified in the project's lock artifact, and no undeclared dependencies are present.

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures block merge to main branch when static analysis detects import path violations or missing error handlers in stream processing code. Code review feedback requires authors to add structured logging or refactor direct service client instantiation before approval. Dependency version mismatches trigger automated alerts and block deployment until resolved.
</enforcement>