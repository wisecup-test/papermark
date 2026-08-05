# Define Service Boundaries Through Explicit Task Definitions: Background Operations That

These rules are ALWAYS ACTIVE for all background operations that coordinate multiple subsystems, including storage, database persistence, event streaming, and external service invocation.

### Rules

- **R-SVCBND-001** MUST: Background operations that coordinate multiple subsystems MUST be defined as discrete task definitions with explicit payload contracts.

### Verify

```bash
# Discover the project's task execution framework configuration and locate the task registry or definition index
find . -type f -name "*.ts" -o -name "*.js" | xargs grep -l "@trigger.dev" | head -5

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
Claude Code MUST NOT skip or defer verification. All background operations coordinating multiple subsystems require explicit task definitions with typed payload contracts before code review approval.
</enforcement>