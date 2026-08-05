# Adopt Modular Core Library Architecture with Observability and Event-Driven Boundaries: Data Integrity Verification

These rules are ALWAYS ACTIVE for feature modules within the enterprise edition that coordinate external services, long-running workflows that require operational visibility and progress tracking, data transformation pipelines that process user-generated content or sensitive data, and API task definitions that expose asynchronous operations to orchestration layers.

### Rules

- **R-MODULAR-001** MUST: Data integrity verification MUST be implemented using middleware-style stream transformations that compute hashes or checksums inline with data flow.
- **R-MODULAR-002** MUST: Feature modules MUST import core infrastructure libraries from the established internal library path, not directly instantiate external service clients.
- **R-MODULAR-003** MUST: Long-running workflows MUST emit structured log events at key lifecycle points with required contextual metadata fields.
- **R-MODULAR-004** MUST: Event-driven stream processing MUST register error handlers at each transformation stage to provide granular failure diagnostics.
- **R-MODULAR-005** MUST: Cache metadata schemas for workflows requiring progress tracking MUST include both numeric progress indicators and human-readable status text, with idempotent progress updates.
- **R-MODULAR-006** SHOULD: Stream-based transformations SHOULD use the pipeline pattern to compose multiple transformation stages.
- **R-MODULAR-007** SHOULD: Error handlers in event-driven boundaries SHOULD either propagate errors to upstream handlers or emit metrics that trigger alerting.

### Verify

```bash
# Discover the project's static analysis configuration and execute linting rules
# that enforce import path conventions for core library usage
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.pylintrc' -o -name 'pyproject.toml' | head -1

# Locate the project's test suite directory and run integration tests
# that verify structured logging output contains required contextual metadata fields
find . -type d -name '__tests__' -o -name 'tests' -o -name 'test' | head -1

# Identify the project's dependency verification script and execute it
# to confirm that all versioned libraries match the resolved versions in the lock artifact
find . -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1
```

**Accept when:**
- Static analysis passes with no violations of core library import path conventions, confirming that feature modules do not directly instantiate external service clients.
- Integration tests confirm that all long-running workflows emit structured log events at key lifecycle points with the required contextual metadata fields.
- Dependency verification confirms that the runtime environment matches the exact versions specified in the project's lock artifact, and no undeclared dependencies are present.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code within scope.
</enforcement>