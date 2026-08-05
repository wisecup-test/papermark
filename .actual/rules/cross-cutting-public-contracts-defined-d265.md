# Adopt Modular Core Library Architecture with Observability and Event-Driven Boundaries: Public Contracts Defined

These rules are ALWAYS ACTIVE for feature modules within the enterprise edition that coordinate external services, long-running workflows that require operational visibility and progress tracking, data transformation pipelines that process user-generated content or sensitive data, and API task definitions that expose asynchronous operations to orchestration layers.

### Rules

- **R-MODULAR-001** SHOULD: Public API contracts SHOULD be defined as typed payload interfaces and exported task definitions to establish clear boundaries between feature modules and orchestration layers.
- **R-MODULAR-002** SHOULD: Establish a consistent internal import pattern across feature modules to signal architectural boundaries and centralize infrastructure concerns.
- **R-MODULAR-003** SHOULD: Design cache metadata schemas upfront to include both numeric progress indicators and human-readable status text, ensuring that progress updates are idempotent to handle retry scenarios.
- **R-MODULAR-004** SHOULD: Use the pipeline pattern to compose multiple transformation stages in stream-based transformations, ensuring that each stage registers its own error handler to provide granular failure diagnostics.
- **R-MODULAR-005** MUST: Implement structured logging with contextual metadata at key lifecycle points in long-running workflows to provide operational visibility.
- **R-MODULAR-006** MUST: Establish error handlers in event-driven boundaries that either propagate errors to upstream handlers or emit metrics that trigger alerting, ensuring that failures are visible to operations teams.
- **R-MODULAR-007** MUST: Implement log sanitization middleware that redacts sensitive fields before emission to prevent inadvertent logging of sensitive user data.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting rules
# that enforce import path conventions for core library usage.
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.pylintrc' -o -name 'pyproject.toml' | head -1

# Locate the project's test suite directory and run integration tests that verify
# structured logging output contains required contextual metadata fields.
find . -type d -name '__tests__' -o -name 'tests' -o -name 'test' | head -1

# Identify the project's dependency verification script and execute it to confirm
# that all versioned libraries match the resolved versions in the lock artifact.
find . -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1
```

**Accept when:**
- Static analysis passes with no violations of core library import path conventions, confirming that feature modules do not directly instantiate external service clients.
- Integration tests confirm that all long-running workflows emit structured log events at key lifecycle points with the required contextual metadata fields.
- Dependency verification confirms that the runtime environment matches the exact versions specified in the project's lock artifact, and no undeclared dependencies are present.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code that falls within the defined scope.
</enforcement>