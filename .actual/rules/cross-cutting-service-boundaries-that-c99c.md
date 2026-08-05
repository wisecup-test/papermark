# Isolate Asynchronous Message Operations Behind Dedicated Service Boundaries: Service Boundaries That

These rules are ALWAYS ACTIVE for HTTP route handlers that invoke asynchronous message operations, service modules that encapsulate message queue client libraries, API endpoints that coordinate database persistence with message delivery, and error handling logic for both synchronous validation and asynchronous delivery failures.

### Rules

- **R-MSG-001** MUST: Service boundaries that coordinate message queue operations must accept domain-level parameters and must not expose queue-specific configuration, connection details, or protocol semantics to calling code.

### Verify

```bash
# Discover the project's dependency manifest and identify the build tool
find . -maxdepth 2 -type f \( -name 'package.json' -o -name 'pom.xml' -o -name 'build.gradle' -o -name 'Gemfile' -o -name 'go.mod' -o -name 'Cargo.toml' \) | head -1

# Inspect the lock or resolution artifact to determine exact versions of messaging client libraries
find . -maxdepth 2 -type f \( -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' -o -name 'Gemfile.lock' -o -name 'go.sum' -o -name 'Cargo.lock' \) | head -1

# Locate test suite configuration
find . -maxdepth 2 -type f \( -name 'jest.config.*' -o -name 'vitest.config.*' -o -name 'pytest.ini' -o -name '.rspec' -o -name 'Makefile' \) | head -1

# Search for route handler implementations that invoke message queue operations
grep -r "queue\|message\|enqueue\|publish" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" --include="*.rb" . 2>/dev/null | grep -E "(route|handler|endpoint)" | head -20

# Verify service boundaries exist and abstract messaging details
grep -r "service.*message\|message.*service" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" --include="*.rb" . 2>/dev/null | head -20

# Check for direct message queue client library imports in route handlers
grep -r "import.*queue\|import.*amqp\|import.*kafka\|import.*redis" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" --include="*.rb" . 2>/dev/null | grep -E "(route|handler|endpoint)" | head -20
```

**Accept when:**
- All route handlers that invoke message queue operations do so through service boundaries that accept domain-level parameters and abstract messaging implementation details.
- Test suite execution confirms that service boundaries can be tested with mock implementations without requiring external messaging infrastructure.
- Error handling distinguishes between synchronous validation failures that block HTTP responses and asynchronous delivery failures that are logged with correlation identifiers.
- No direct imports of message queue client libraries appear in route handler modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers invoking message queue operations MUST use service boundaries. Direct client library calls in route handlers are violations requiring refactoring.
</enforcement>