# Define Service Boundaries Through Explicit Task Definitions: Task Implementations That

These rules are ALWAYS ACTIVE for all background operations requiring multi-step coordination across multiple subsystems, asynchronous workflows with progress tracking, operations spanning multiple subsystem boundaries, and long-running tasks requiring independent scaling and monitoring.

### Rules

- **R-TASK-001** MUST: Task implementations that depend on versioned SDK libraries MUST discover the exact resolved version from the project's lock artifact before implementation.
- **R-TASK-002** MUST: All background operations with multi-subsystem coordination MUST be defined as discrete tasks with exported payload contracts.
- **R-TASK-003** MUST: Task implementations MUST include structured logging with contextual metadata at key execution points.
- **R-TASK-004** MUST: Task definitions MUST be colocated with their domain logic but exported through a public contract module to establish clear boundaries.
- **R-TASK-005** SHOULD: Progress tracking through metadata caching SHOULD use normalized progress values between 0 and 1 with human-readable status text.
- **R-TASK-006** SHOULD: Event handlers for subsystem coordination SHOULD implement proper error propagation and cleanup to prevent resource leaks.

### Verify

```bash
# Discover the project's task execution framework configuration and locate the task registry or definition index
find . -name "*.config.*" -o -name "*registry*" | grep -E "(task|trigger)" | head -20

# Identify the project's testing framework and execute tests that validate task payload contracts and execution boundaries
grep -r "describe\|test\|it(" --include="*.test.*" --include="*.spec.*" | grep -i "task\|payload\|contract" | head -20

# Locate the project's logging configuration and verify structured logging output includes required contextual metadata fields
grep -r "logger\|log\|structured" --include="*.ts" --include="*.js" | grep -E "(context|metadata|trace)" | head -20

# Verify task definitions export required payload contracts
grep -r "export.*task\|export.*payload" --include="*.ts" --include="*.js" | head -20

# Check lock file for exact resolved versions of SDK dependencies
cat package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | grep -A 2 "trigger\|sdk" | head -30
```

**Accept when:**
- All background operations with multi-subsystem coordination are defined as discrete tasks with exported payload contracts
- Task implementations include structured logging with contextual metadata at key execution points
- Tests validate task execution boundaries and payload contract compliance
- Exact resolved versions of all versioned SDK libraries are discovered from the project's lock artifact before implementation
- Task definitions are colocated with domain logic and exported through public contract modules
- Event handlers implement proper error propagation and cleanup

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are non-negotiable. Code review and static analysis verification are mandatory before acceptance.
</enforcement>