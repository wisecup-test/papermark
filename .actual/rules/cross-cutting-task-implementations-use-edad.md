# Define Service Boundaries Through Explicit Task Definitions: Task Implementations Use

These rules are ALWAYS ACTIVE for all background operations requiring multi-step coordination across multiple subsystems, asynchronous workflows with progress tracking requirements, operations spanning multiple subsystem boundaries, and long-running tasks requiring independent scaling and monitoring.

### Rules

- **R-EDAD-001** SHOULD: Task implementations SHOULD use metadata caching mechanisms to communicate progress state to external observers.

### Verify

```bash
# Discover the project's task execution framework configuration and locate the task registry or definition index
find . -type f -name '*.json' -o -name '*.yaml' -o -name '*.yml' | xargs grep -l 'task\|trigger' | head -5

# Identify the project's testing framework and execute tests that validate task payload contracts and execution boundaries
find . -type f \( -name '*.test.*' -o -name '*.spec.*' \) | grep -i task | head -10

# Locate the project's logging configuration and verify structured logging output includes required contextual metadata fields
find . -type f -name '*.ts' -o -name '*.js' | xargs grep -l 'structured.*log\|logger.*context\|metadata' | head -10
```

**Accept when:**
- All background operations with multi-subsystem coordination are defined as discrete tasks with exported payload contracts
- Task implementations include structured logging with contextual metadata at key execution points
- Tests validate task execution boundaries and payload contract compliance

<enforcement>
Claude Code MUST NOT skip or defer verification. All background operations matching the scope criteria MUST comply with R-EDAD-001 or document an approved exception with architecture review board rationale.
</enforcement>