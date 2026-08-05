# Adopt Modular Core Library Architecture with Observability and Event-Driven Boundaries: Before Any Versioned

These rules are ALWAYS ACTIVE for feature modules within the enterprise edition that coordinate external services, long-running workflows that require operational visibility and progress tracking, data transformation pipelines that process user-generated content or sensitive data, and API task definitions that expose asynchronous operations to orchestration layers.

### Rules

- **R-MODULAR-001** MUST: Before using any versioned library, implementations MUST discover the project's dependency lock artifact, resolve the exact installed version, and verify API compatibility against that version's official documentation.
- **R-MODULAR-002** MUST: Feature modules MUST use internal library abstractions for infrastructure concerns (storage configuration, AWS SDK clients, database access) rather than directly instantiating external service clients.
- **R-MODULAR-003** MUST: Long-running workflows MUST emit structured log events at key lifecycle points with required contextual metadata fields.
- **R-MODULAR-004** MUST: Event-driven stream processing implementations MUST register explicit error handlers at each transformation stage to provide granular failure diagnostics.
- **R-MODULAR-005** MUST: Cache metadata schemas for workflows requiring progress tracking MUST include both numeric progress indicators and human-readable status text, with idempotent progress updates.
- **R-MODULAR-006** SHOULD: Data transformation pipelines SHOULD use the pipeline pattern to compose multiple transformation stages with isolated error handling.
- **R-MODULAR-007** SHOULD: Structured logging implementations SHOULD include log sanitization middleware to redact sensitive fields before emission.
- **R-MODULAR-008** MAY: Developers MAY request exceptions to import path conventions by documenting technical justification in an architecture decision log entry and obtaining platform engineering lead approval.

### Verify

```bash
# Discover the project's static analysis configuration and execute linting rules
# that enforce import path conventions for core library usage
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.pylintrc' -o -name 'pyproject.toml' | head -1

# Locate the project's test suite directory and run integration tests
# that verify structured logging output contains required contextual metadata fields
find . -type d -name '__tests__' -o -name 'tests' -o -name 'test' | head -1

# Identify the project's dependency lock artifact
find . -maxdepth 2 -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' -o -name 'Pipfile.lock' -o -name 'poetry.lock' | head -1

# Verify dependency versions match lock artifact
grep -r 'require\|import' . --include='*.js' --include='*.ts' --include='*.py' | grep -v node_modules | head -20
```

**Accept when:**
- Static analysis passes with no violations of core library import path conventions, confirming that feature modules do not directly instantiate external service clients.
- Integration tests confirm that all long-running workflows emit structured log events at key lifecycle points with the required contextual metadata fields.
- Dependency verification confirms that the runtime environment matches the exact versions specified in the project's lock artifact, and no undeclared dependencies are present.
- All event-driven stream processing code includes explicit error handlers at each transformation stage.
- Cache metadata schemas include both numeric progress indicators and human-readable status text with idempotent update semantics.

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-MODULAR rules marked MUST are mandatory and must be verified before code acceptance. Violations block merge to main branch.
</enforcement>