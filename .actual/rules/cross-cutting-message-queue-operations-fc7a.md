# Isolate Asynchronous Message Operations Behind Dedicated Service Boundaries: Message Queue Operations

These rules are ALWAYS ACTIVE for all HTTP route handlers that invoke asynchronous message queue operations, service modules that encapsulate message queue client libraries, and API endpoints that coordinate database persistence with message delivery.

### Rules

- **R-MSG-001** MUST: All message queue operations invoked from HTTP route handlers must be encapsulated behind dedicated service boundaries that abstract the messaging implementation from transport-layer concerns.
- **R-MSG-002** MUST: Service boundaries must accept domain-level parameters that describe message content and routing requirements without exposing queue names, connection strings, or protocol-specific configuration.
- **R-MSG-003** MUST: Error handling must distinguish between validation failures that occur before message enqueuing (which block HTTP responses) and delivery failures that occur asynchronously (which are logged with correlation identifiers).
- **R-MSG-004** SHOULD: Implement service boundaries as dependency-injected interfaces to enable testing with in-memory implementations and production deployment with distributed queue clients.
- **R-MSG-005** SHOULD: Service boundary implementations must expose sufficient observability to diagnose asynchronous delivery failures and document delivery semantics, retry policies, and failure modes.

### Verify

```bash
# Discover the project's dependency manifest and identify the build tool
find . -maxdepth 2 -type f \( -name 'package.json' -o -name 'pom.xml' -o -name 'build.gradle' -o -name 'Gemfile' -o -name 'go.mod' -o -name 'Cargo.toml' \) | head -1

# Inspect the lock or resolution artifact to determine exact versions of messaging client libraries
find . -maxdepth 2 -type f \( -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' -o -name 'Gemfile.lock' -o -name 'go.sum' -o -name 'Cargo.lock' \) | head -1

# Search for route handler files that may invoke message queue operations
grep -r "route\|handler\|endpoint" --include="*.js" --include="*.ts" --include="*.py" --include="*.go" --include="*.java" . 2>/dev/null | grep -E "(app\.|router\.|@|def |func )" | head -20

# Detect direct message queue client library imports in route handler modules
grep -r "import.*queue\|import.*amqp\|import.*kafka\|import.*sqs\|import.*pubsub\|require.*queue\|require.*amqp\|require.*kafka" --include="*.js" --include="*.ts" --include="*.py" --include="*.go" --include="*.java" . 2>/dev/null | grep -v node_modules | grep -v ".actual"

# Locate test suite configuration
find . -maxdepth 2 -type f \( -name 'jest.config.*' -o -name 'pytest.ini' -o -name 'vitest.config.*' -o -name '.mocharc.*' -o -name 'karma.conf.*' \) | head -1

# Search for service boundary implementations that abstract message queue operations
grep -r "service\|Service" --include="*.js" --include="*.ts" --include="*.py" --include="*.go" --include="*.java" . 2>/dev/null | grep -E "(message|queue|async)" | grep -v node_modules | grep -v ".actual" | head -20
```

**Accept when:**
- All route handlers that invoke message queue operations do so through service boundaries that accept domain-level parameters and abstract messaging implementation details.
- Test suite execution confirms that service boundaries can be tested with mock implementations without requiring external messaging infrastructure.
- Error handling distinguishes between synchronous validation failures that block HTTP responses and asynchronous delivery failures that are logged with correlation identifiers.
- No direct imports of message queue client libraries are found in route handler modules; all such imports are confined to dedicated service boundary modules.
- Service boundary interfaces are dependency-injected and support both in-memory test implementations and production queue clients.

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers invoking message queue operations MUST be refactored to use service boundaries before code review approval.
</enforcement>