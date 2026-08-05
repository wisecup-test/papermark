# Isolate Asynchronous Message Operations Behind Dedicated Service Boundaries: Message Queue Service

These rules are ALWAYS ACTIVE for HTTP route handlers that invoke asynchronous message operations, service modules that encapsulate message queue client libraries, API endpoints that coordinate database persistence with message delivery, and error handling logic for both synchronous validation and asynchronous delivery failures.

### Rules

- **R-MSG-001** SHOULD: Message queue service boundaries should provide structured logging or telemetry that correlates asynchronous operations with the originating request context for debugging and tracing.

### Verify

```bash
# Discover the project's dependency manifest and identify the build tool
find . -maxdepth 2 -type f \( -name 'package.json' -o -name 'pom.xml' -o -name 'build.gradle' -o -name 'Gemfile' -o -name 'go.mod' -o -name 'Cargo.toml' \) | head -1

# Inspect the lock or resolution artifact to determine exact versions of messaging client libraries
find . -maxdepth 2 -type f \( -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' -o -name '.lock' -o -name 'Gemfile.lock' -o -name 'go.sum' -o -name 'Cargo.lock' \) | head -1

# Locate test suite configuration
find . -maxdepth 3 -type f \( -name 'jest.config.*' -o -name 'vitest.config.*' -o -name 'pytest.ini' -o -name 'rspec.rb' -o -name 'go.test' \) | head -1

# Search for route handlers that invoke message queue operations
grep -r "enqueue\|publish\|send.*message\|queue\." --include="*.js" --include="*.ts" --include="*.py" --include="*.go" --include="*.rb" . 2>/dev/null | grep -E "(route|handler|endpoint)" | head -20

# Verify service boundaries exist by searching for service/queue module patterns
find . -path ./node_modules -prune -o -type f \( -name '*service*.js' -o -name '*service*.ts' -o -name '*queue*.js' -o -name '*queue*.ts' \) -print | grep -v node_modules | head -20

# Check for direct message queue client imports in route handlers
grep -r "from.*queue\|import.*queue\|require.*queue" --include="*.js" --include="*.ts" --include="*.py" . 2>/dev/null | grep -E "(route|handler|endpoint)" | head -10
```

**Accept when:**
- All route handlers that invoke message queue operations do so through service boundaries that accept domain-level parameters and abstract messaging implementation details.
- Test suite execution confirms that service boundaries can be tested with mock implementations without requiring external messaging infrastructure.
- Error handling distinguishes between synchronous validation failures that block HTTP responses and asynchronous delivery failures that are logged with correlation identifiers.
- Service boundaries provide structured logging or telemetry that correlates asynchronous operations with originating request context.

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers invoking message queue operations MUST use service boundaries. Direct message queue client library calls in route handlers are violations requiring refactoring.
</enforcement>