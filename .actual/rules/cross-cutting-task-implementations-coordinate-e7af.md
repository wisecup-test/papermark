# Define Service Boundaries Through Explicit Task Definitions: Task Implementations Coordinate

These rules are ALWAYS ACTIVE for all background operations requiring multi-step coordination across multiple subsystems, asynchronous workflows with progress tracking, operations spanning multiple subsystem boundaries, and long-running tasks requiring independent scaling and monitoring.

### Rules

- **R-TASK-001** SHOULD: Task implementations SHOULD coordinate subsystem interactions through event-driven patterns rather than direct coupling.

### Verify

```bash
# Discover the project's task execution framework configuration and locate the task registry or definition index
find . -name "*.ts" -o -name "*.js" | xargs grep -l "@trigger\.dev" | head -5

# Identify the project's testing framework and execute tests that validate task payload contracts and execution boundaries
grep -r "describe\|test\|it(" --include="*.test.ts" --include="*.spec.ts" | grep -i "task\|payload" | head -10

# Locate the project's logging configuration and verify structured logging output includes required contextual metadata fields
grep -r "logger\|log\|winston\|pino" --include="*.ts" --include="*.js" | grep -i "context\|metadata" | head -10
```

**Accept when:**
- All background operations with multi-subsystem coordination are defined as discrete tasks with exported payload contracts
- Task implementations include structured logging with contextual metadata at key execution points
- Tests validate task execution boundaries and payload contract compliance
- Task definitions are colocated with domain logic but exported through a public contract module
- Progress tracking uses normalized progress values between 0 and 1 with human-readable status text
- Event handlers implement proper error propagation and cleanup to prevent resource leaks

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review verification that background operations follow task definition pattern is mandatory. Static analysis verification that task definitions export required payload contracts is mandatory. Integration tests validating task execution and observability requirements are mandatory.
</enforcement>