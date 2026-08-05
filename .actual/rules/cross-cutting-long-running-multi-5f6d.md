# Adopt Modular Core Library Architecture with Observability and Event-Driven Boundaries: Long Running Multi

These rules are ALWAYS ACTIVE for feature modules within the enterprise edition that coordinate external services, long-running workflows that require operational visibility and progress tracking, data transformation pipelines that process user-generated content or sensitive data, and API task definitions that expose asynchronous operations to orchestration layers.

### Rules

- **R-MODULAR-001** MUST: All long-running or multi-step workflows MUST emit structured log events with contextual metadata at key lifecycle points to enable operational observability.
- **R-MODULAR-002** MUST: Feature modules MUST import core infrastructure libraries from the established internal library path, not directly instantiate external service clients.
- **R-MODULAR-003** MUST: Event-driven stream processing implementations MUST register explicit error handlers at each transformation stage to provide granular failure diagnostics.
- **R-MODULAR-004** MUST: Cache metadata schemas for workflows requiring progress tracking MUST include both numeric progress indicators and human-readable status text, with idempotent update semantics.
- **R-MODULAR-005** SHOULD: Stream-based transformations SHOULD use the pipeline pattern to compose multiple transformation stages with isolated error handling.
- **R-MODULAR-006** SHOULD: Progress updates in long-running workflows SHOULD be designed to handle retry scenarios through idempotent state transitions.
- **R-MODULAR-007** MAY: External APM agents MAY be used as a complementary approach to capture low-level performance metrics and distributed traces alongside domain-specific structured logs.

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
Claude Code MUST NOT skip or defer verification. All three acceptance criteria MUST be satisfied before approving code that implements long-running workflows or modifies core infrastructure library patterns.
</enforcement>