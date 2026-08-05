# Define Service Boundaries Through Explicit Task Definitions: Task Implementations Use

These rules are ALWAYS ACTIVE for all background operations requiring multi-step coordination across multiple subsystems, asynchronous workflows with progress tracking, and long-running tasks requiring independent scaling and monitoring.

### Rules

- **R-TASK-001** MUST: Task implementations MUST use structured logging with contextual metadata to enable observability across service boundaries.

### Verify

```bash
# Discover the project's task execution framework configuration and locate the task registry or definition index
find . -type f -name "*.ts" -o -name "*.js" | xargs grep -l "@trigger\.dev" | head -5

# Identify the project's testing framework and execute tests that validate task payload contracts and execution boundaries
grep -r "describe\|test\|it(" --include="*.test.ts" --include="*.spec.ts" | grep -i "task\|payload" | head -10

# Locate the project's logging configuration and verify structured logging output includes required contextual metadata fields
find . -type f \( -name "*.ts" -o -name "*.js" \) | xargs grep -l "structured.*log\|logger.*context\|metadata" | head -5
```

**Accept when:**
- All background operations with multi-subsystem coordination are defined as discrete tasks with exported payload contracts
- Task implementations include structured logging with contextual metadata at key execution points
- Tests validate task execution boundaries and payload contract compliance

<enforcement>
Claude Code MUST NOT skip or defer verification. All background operations must be reviewed for compliance with task definition patterns and structured logging requirements before approval.
</enforcement>