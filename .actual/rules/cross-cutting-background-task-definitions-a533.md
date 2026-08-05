# Export Typed Task Payloads as Public API Contracts: Background Task Definitions

These rules are ALWAYS ACTIVE for all background task definitions that coordinate multi-step workflows, span process or runtime boundaries, require progress tracking or observability correlation, or are consumed by external orchestration systems or SDKs.

### Rules

- **R-TASK-001** MUST: All background task definitions SHALL export a typed payload interface and a task identifier as public API contracts.
- **R-TASK-002** MUST: Define task payload interfaces in dedicated contract modules separate from implementation logic to enable independent testing and versioning.
- **R-TASK-003** MUST: Export both the payload type and task identifier as public API.
- **R-TASK-004** MUST: Implement runtime validation at task entry points to verify payloads conform to exported contracts.
- **R-TASK-005** MUST: Use schema validation to catch type drift and provide clear error messages for contract violations.
- **R-TASK-006** SHOULD: Structure task implementations to emit observability events at workflow stage boundaries using consistent metadata shapes derived from the task payload.
- **R-TASK-007** SHOULD: Ensure correlation identifiers flow through all subsystem interactions.

### Verify

```bash
# Discover the project's type-checking configuration and execute the type checker
# to verify all task payload exports are well-typed and all task invocations pass type validation
type_checker_config=$(find . -name "tsconfig.json" -o -name "pyproject.toml" -o -name ".eslintrc*" | head -1)
if [ -n "$type_checker_config" ]; then
  echo "Found type checker config: $type_checker_config"
  # Execute type checker (tool-specific command derived from project)
fi

# Locate the project's test suite and execute contract tests
# that verify task payloads conform to exported type definitions
test_dir=$(find . -type d -name "test*" -o -name "__tests__" -o -name "spec" | head -1)
if [ -n "$test_dir" ]; then
  echo "Found test directory: $test_dir"
  # Execute contract tests (tool-specific command derived from project)
fi

# Identify the project's linting or static analysis tooling
# and verify it detects untyped task payloads or missing contract exports
lint_config=$(find . -name ".eslintrc*" -o -name "pylintrc" -o -name "setup.cfg" | head -1)
if [ -n "$lint_config" ]; then
  echo "Found lint config: $lint_config"
  # Execute linter (tool-specific command derived from project)
fi
```

**Accept when:**
- Type checker reports no errors for task payload definitions and all task invocation sites pass type validation
- Contract tests verify runtime validation rejects payloads that violate exported type constraints
- Static analysis confirms all background task definitions export typed payload interfaces and task identifiers as public API
- All background task definitions in scope have dedicated contract modules with exported payload types and task identifiers
- Runtime validation is implemented at all task entry points

<enforcement>
Claude Code MUST NOT skip or defer verification. Type check failures, contract test failures, or missing contract exports block merge to protected branches and prevent deployment. Code review process requires explicit approval for task contract changes. Violations require documented rationale in architectural decision log and approval from technical lead or architecture review board.
</enforcement>