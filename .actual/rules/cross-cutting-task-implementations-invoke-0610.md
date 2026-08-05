# Define Service Boundaries Through Explicit Task Definitions: Task Implementations Invoke

These rules are ALWAYS ACTIVE for all background operations requiring multi-step coordination across multiple subsystems, asynchronous workflows with progress tracking, and long-running tasks spanning service boundaries.

### Rules

- **R-TASK-001** MAY: Task implementations MAY invoke external services through client SDK abstractions when cross-service coordination is required.
- **R-TASK-002** MUST: All background operations with multi-subsystem coordination be defined as discrete tasks with exported payload contracts.
- **R-TASK-003** MUST: Task implementations include structured logging with contextual metadata at key execution points.
- **R-TASK-004** MUST: Task definitions be colocated with their domain logic but exported through a public contract module to establish clear boundaries.
- **R-TASK-005** SHOULD: Progress tracking through metadata caching use normalized progress values between 0 and 1 with human-readable status text.
- **R-TASK-006** SHOULD: Event handlers for subsystem coordination implement proper error propagation and cleanup to prevent resource leaks.
- **R-TASK-007** MUST NOT: Synchronous request-response handlers be implemented as task definitions.
- **R-TASK-008** MUST NOT: Single-subsystem operations without coordination requirements be implemented as task definitions.

### Verify

```bash
# Discover the project's task execution framework configuration and locate the task registry or definition index
find . -type f -name '*.ts' -o -name '*.js' | xargs grep -l '@trigger\.dev\|task.*definition\|taskRegistry' | head -20

# Identify the project's testing framework and execute tests that validate task payload contracts and execution boundaries
grep -r 'describe\|test\|it(' . --include='*.test.ts' --include='*.spec.ts' | grep -i 'task\|payload\|contract' | head -20

# Locate the project's logging configuration and verify structured logging output includes required contextual metadata fields
find . -type f \( -name '*.ts' -o -name '*.js' \) | xargs grep -l 'logger\|log\|winston\|pino' | xargs grep -l 'metadata\|context\|traceId' | head -20

# Verify task definitions export required payload contracts
find . -type f -name '*.ts' | xargs grep -l 'export.*task\|export.*payload' | head -20
```

**Accept when:**
- All background operations with multi-subsystem coordination are defined as discrete tasks with exported payload contracts
- Task implementations include structured logging with contextual metadata at key execution points
- Tests validate task execution boundaries and payload contract compliance
- Task definitions are colocated with domain logic and exported through public contract modules
- Progress tracking uses normalized values between 0 and 1 with human-readable status text
- Event handlers implement proper error propagation and cleanup

<enforcement>
Clause MUST NOT skip or defer verification. Code review verification that background operations follow task definition pattern is mandatory. Static analysis verification that task definitions export required payload contracts is mandatory. Integration tests validating task execution and observability requirements are mandatory.
</enforcement>