# Define Service Boundaries Through Explicit Task Definitions: Task Definitions Export

These rules are ALWAYS ACTIVE for all background operations requiring multi-step coordination across multiple subsystems, asynchronous workflows with progress tracking, operations spanning multiple subsystem boundaries, and long-running tasks requiring independent scaling and monitoring.

### Rules

- **R-TASK-001** MUST: Task definitions MUST export both the payload type contract and the task identifier to establish service boundaries.
- **R-TASK-002** MUST: Task definitions should be colocated with their domain logic but exported through a public contract module to establish clear boundaries.
- **R-TASK-003** MUST: Progress tracking through metadata caching should use normalized progress values between 0 and 1 with human-readable status text.
- **R-TASK-004** MUST: Event handlers for subsystem coordination should implement proper error propagation and cleanup to prevent resource leaks.
- **R-TASK-005** MUST: Task implementations include structured logging with contextual metadata at key execution points.

### Verify

```bash
# Discover the project's task execution framework configuration and locate the task registry or definition index
find . -type f -name "*.ts" -o -name "*.js" | xargs grep -l "@trigger\.dev" | head -5

# Identify the project's testing framework and execute tests that validate task payload contracts and execution boundaries
grep -r "describe\|test\|it(" --include="*.test.ts" --include="*.spec.ts" | grep -i "task\|payload" | head -10

# Locate the project's logging configuration and verify structured logging output includes required contextual metadata fields
grep -r "logger\|log\|winston\|pino" --include="*.ts" --include="*.js" | grep -i "context\|metadata" | head -10

# Verify task definitions export payload contracts
grep -r "export.*type.*Payload\|export.*interface.*Payload" --include="*.ts" | head -10

# Verify task identifier exports
grep -r "export.*const.*task\|export.*const.*Task" --include="*.ts" | grep -v "test\|spec" | head -10
```

**Accept when:**
- All background operations with multi-subsystem coordination are defined as discrete tasks with exported payload contracts
- Task implementations include structured logging with contextual metadata at key execution points
- Tests validate task execution boundaries and payload contract compliance
- Task definitions export both payload type contracts and task identifiers
- Progress tracking uses normalized values (0-1) with human-readable status text
- Event handlers implement proper error propagation and cleanup

<enforcement>
Claude Code MUST NOT skip or defer verification. All background operations must be reviewed against R-TASK-001 through R-TASK-005 before approval.
</enforcement>