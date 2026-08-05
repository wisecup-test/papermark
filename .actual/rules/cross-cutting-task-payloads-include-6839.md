# Export Typed Task Payloads as Public API Contracts: Task Payloads Include

These rules are ALWAYS ACTIVE for all background task definitions that coordinate multi-step workflows, span process or runtime boundaries, or are invoked across asynchronous execution contexts.

### Rules

- **R-TASK-001** SHOULD: Task payloads SHOULD include identifiers sufficient for correlation with observability logs and database records.
- **R-TASK-002** MUST: Define task payload interfaces in dedicated contract modules separate from implementation logic to enable independent testing and versioning.
- **R-TASK-003** MUST: Export both the payload type and task identifier as public API.
- **R-TASK-004** MUST: Implement runtime validation at task entry points to verify payloads conform to exported contracts.
- **R-TASK-005** MUST: Use schema validation to catch type drift and provide clear error messages for contract violations.
- **R-TASK-006** MUST: Structure task implementations to emit observability events at workflow stage boundaries using consistent metadata shapes derived from the task payload.
- **R-TASK-007** MUST: Ensure correlation identifiers flow through all subsystem interactions.

### Verify

```bash
# Discover the project's type-checking configuration and execute the type checker
# to verify all task payload exports are well-typed and all task invocations pass type validation
type_checker_config=$(find . -name "tsconfig.json" -o -name "pyproject.toml" -o -name ".eslintrc*" | head -1)
if [ -n "$type_checker_config" ]; then
  echo "Found type checker config: $type_checker_config"
  # Execute type checker (tool discovery from project manifest)
fi

# Locate the project's test suite and execute contract tests
# that verify task payloads conform to exported type definitions
test_dir=$(find . -type d -name "test*" -o -name "__tests__" -o -name "spec" | head -1)
if [ -n "$test_dir" ]; then
  echo "Found test directory: $test_dir"
  # Execute contract tests (tool discovery from project manifest)
fi

# Identify the project's linting or static analysis tooling
# and verify it detects untyped task payloads or missing contract exports
lint_config=$(find . -name ".eslintrc*" -o -name "pylintrc" -o -name "setup.cfg" | head -1)
if [ -n "$lint_config" ]; then
  echo "Found lint config: $lint_config"
  # Execute static analysis (tool discovery from project manifest)
fi
```

**Accept when:**
- Type checker reports no errors for task payload definitions and all task invocation sites pass type validation
- Contract tests verify runtime validation rejects payloads that violate exported type constraints
- Static analysis confirms all background task definitions export typed payload interfaces and task identifiers as public API
- All task payloads include correlation identifiers (e.g., request ID, trace ID, user ID) suitable for observability instrumentation
- Runtime validation is enforced at task entry points with clear error messages for contract violations

<enforcement>
Claude Code MUST NOT skip or defer verification. Type check failures, contract test failures, or missing public API exports block acceptance. Code review MUST verify task contract changes explicitly. Exceptions require documented rationale and technical lead approval.
</enforcement>