# Export Typed Task Payloads as Public API Contracts: Task Implementations That

These rules are ALWAYS ACTIVE for all background task definitions that coordinate multi-step workflows, span process or runtime boundaries, require progress tracking or observability correlation, or are consumed by external orchestration systems or SDKs.

### Rules

- **R-TASK-001** MUST: Task implementations that depend on versioned SDK libraries SHALL require consumers to discover the exact resolved version from the project's lock artifact before integration.
- **R-TASK-002** MUST: Define task payload interfaces in dedicated contract modules separate from implementation logic to enable independent testing and versioning; export both the payload type and task identifier as public API.
- **R-TASK-003** MUST: Implement runtime validation at task entry points to verify payloads conform to exported contracts; use schema validation to catch type drift and provide clear error messages for contract violations.
- **R-TASK-004** MUST: Structure task implementations to emit observability events at workflow stage boundaries using consistent metadata shapes derived from the task payload; ensure correlation identifiers flow through all subsystem interactions.
- **R-TASK-005** SHOULD: Maintain backward compatibility or provide migration paths when evolving public task contracts to minimize impact on multiple consumers across runtime environments.

### Verify

```bash
# Discover the project's type-checking configuration and execute the type checker
# to verify all task payload exports are well-typed and all task invocations pass type validation
type_checker_config=$(find . -name 'tsconfig.json' -o -name 'pyproject.toml' -o -name '.eslintrc*' | head -1)
if [ -n "$type_checker_config" ]; then
  echo "Type checker configuration found: $type_checker_config"
  # Execute type checker (command varies by project)
fi

# Locate the project's test suite and execute contract tests
# that verify task payloads conform to exported type definitions
test_dir=$(find . -type d -name 'test*' -o -name '__tests__' -o -name 'spec' | head -1)
if [ -n "$test_dir" ]; then
  echo "Test directory found: $test_dir"
  # Execute contract tests (command varies by project)
fi

# Identify the project's linting or static analysis tooling
# and verify it detects untyped task payloads or missing contract exports
lint_config=$(find . -name '.eslintrc*' -o -name 'pylintrc' -o -name '.flake8' | head -1)
if [ -n "$lint_config" ]; then
  echo "Linting configuration found: $lint_config"
  # Execute linter (command varies by project)
fi
```

**Accept when:**
- Type checker reports no errors for task payload definitions and all task invocation sites pass type validation
- Contract tests verify runtime validation rejects payloads that violate exported type constraints
- Static analysis confirms all background task definitions export typed payload interfaces and task identifiers as public API
- All task contracts are versioned and documented with SDK version requirements
- Breaking changes to public task contracts are tracked and migration paths are provided

<enforcement>
Claude Code MUST NOT skip or defer verification. Type check failures, contract test failures, or missing public API exports block merge to protected branches and prevent deployment. Code review process requires explicit approval for task contract changes. Exceptions require documentation in the architectural decision log and approval from technical lead or architecture review board.
</enforcement>