# Adopt Modular Core Library Architecture with Observability and Event-Driven Boundaries: Database Access Patterns

These rules are ALWAYS ACTIVE for feature modules within the enterprise edition that coordinate external services, long-running workflows that require operational visibility and progress tracking, data transformation pipelines that process user-generated content or sensitive data, and API task definitions that expose asynchronous operations to orchestration layers.

### Rules

- **R-MODULAR-001** MUST: Database access patterns MUST separate read projections from write operations, using explicit select clauses to minimize data transfer and include clauses to control relationship loading.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting rules that enforce import path conventions for core library usage.
# Locate the project's test suite directory and run integration tests that verify structured logging output contains required contextual metadata fields.
# Identify the project's dependency verification script and execute it to confirm that all versioned libraries match the resolved versions in the lock artifact.
```

**Accept when:**
- Static analysis passes with no violations of core library import path conventions, confirming that feature modules do not directly instantiate external service clients.
- Integration tests confirm that all long-running workflows emit structured log events at key lifecycle points with the required contextual metadata fields.
- Dependency verification confirms that the runtime environment matches the exact versions specified in the project's lock artifact, and no undeclared dependencies are present.

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures block merge to main branch when static analysis detects import path violations or missing error handlers in stream processing code. Code review feedback requires authors to add structured logging or refactor direct service client instantiation before approval. Dependency version mismatches trigger automated alerts and block deployment until resolved.
</enforcement>