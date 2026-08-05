# Adopt Modular Core Library Architecture with Observability and Event-Driven Boundaries: Feature Modules Import

These rules are ALWAYS ACTIVE for feature modules within the enterprise edition that coordinate external services, long-running workflows that require operational visibility and progress tracking, data transformation pipelines that process user-generated content or sensitive data, and API task definitions that expose asynchronous operations to orchestration layers.

### Rules

- **R-MODULAR-001** MUST: Feature modules MUST import core infrastructure libraries from the designated internal library path rather than directly instantiating external service clients.
- **R-MODULAR-002** MUST: Long-running workflows MUST emit structured log events at key lifecycle points with required contextual metadata fields.
- **R-MODULAR-003** MUST: Event-driven stream processing MUST register error handlers at each transformation stage to provide granular failure diagnostics.
- **R-MODULAR-004** MUST: Cache metadata schemas for workflows requiring progress tracking MUST include both numeric progress indicators and human-readable status text, with idempotent progress updates.
- **R-MODULAR-005** MUST: Before using any versioned library, the exact resolved version MUST be confirmed from the project's lock artifact, not from training data or build-tool output alone.
- **R-MODULAR-006** SHOULD: Design internal library abstractions with extension points and escape hatches to allow advanced users to access underlying clients when necessary.
- **R-MODULAR-007** SHOULD: Implement log sanitization middleware that redacts sensitive fields before emission to prevent inadvertent logging of sensitive user data.
- **R-MODULAR-008** SHOULD: Establish a convention that error handlers in stream processing MUST either propagate errors to upstream handlers or emit metrics that trigger alerting.

### Verify

```bash
# Discover the project's static analysis configuration and execute linting rules
# that enforce import path conventions for core library usage
find . -name ".eslintrc*" -o -name "pylintrc" -o -name ".flake8" | head -1

# Locate the project's test suite directory and run integration tests
# that verify structured logging output contains required contextual metadata fields
find . -type d -name "test*" -o -name "__tests__" -o -name "spec" | head -1

# Identify the project's dependency verification script and execute it
# to confirm that all versioned libraries match the resolved versions in the lock artifact
find . -name "package-lock.json" -o -name "yarn.lock" -o -name "Pipfile.lock" -o -name "go.sum" | head -1
```

**Accept when:**
- Static analysis passes with no violations of core library import path conventions, confirming that feature modules do not directly instantiate external service clients.
- Integration tests confirm that all long-running workflows emit structured log events at key lifecycle points with the required contextual metadata fields.
- Dependency verification confirms that the runtime environment matches the exact versions specified in the project's lock artifact, and no undeclared dependencies are present.
- All stream processing code includes explicit error handlers at each transformation stage.
- Cache metadata schemas include both numeric progress indicators and human-readable status text with idempotent update semantics.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and merge approval. Violations block merge to main branch. Exceptions require documented technical justification and platform engineering lead approval.
</enforcement>