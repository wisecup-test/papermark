# Isolate Asynchronous Message Operations Behind Dedicated Service Boundaries: Message Queue Service

These rules are ALWAYS ACTIVE for all HTTP route handlers that invoke asynchronous message operations, service modules that encapsulate message queue client libraries, API endpoints that coordinate database persistence with message delivery, and error handling logic for both synchronous validation and asynchronous delivery failures.

### Rules

- **R-MSG-001** MUST: Encapsulate all message queue operations behind dedicated service boundaries that accept domain-level parameters describing message content and routing requirements without exposing queue names, connection strings, or protocol-specific configuration.
- **R-MSG-002** MUST: Implement service boundaries as dependency-injected interfaces to enable testing with in-memory implementations and production deployment with distributed queue clients.
- **R-MSG-003** MUST: Distinguish between validation failures that occur before message enqueuing (which block HTTP responses with appropriate error codes) and delivery failures that occur asynchronously (which are logged with correlation identifiers).
- **R-MSG-004** SHOULD: Support multiple transport backends through adapter patterns to enable testing with in-memory queues and production deployment with distributed messaging systems.
- **R-MSG-005** SHOULD: Document the delivery semantics, retry policies, and failure modes of the service boundary interface to ensure that service implementations expose sufficient observability to diagnose asynchronous delivery failures.
- **R-MSG-006** SHOULD: Adopt transactional outbox pattern or similar techniques to ensure consistency between database writes and message delivery when service boundaries coordinate database transactions with message delivery.

### Verify

```bash
# Discover the project's dependency manifest and identify the build tool
find . -maxdepth 2 -type f \( -name 'package.json' -o -name 'pom.xml' -o -name 'build.gradle' -o -name 'Gemfile' -o -name 'go.mod' -o -name 'Cargo.toml' \) | head -1

# Inspect the lock or resolution artifact to determine exact versions of messaging client libraries
find . -maxdepth 2 -type f \( -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pom.lock' -o -name 'Gemfile.lock' -o -name 'go.sum' -o -name 'Cargo.lock' \) | head -1

# Search for route handler implementations that invoke message queue operations
grep -r "enqueue\|publish\|send.*message" --include="*.js" --include="*.ts" --include="*.py" --include="*.java" --include="*.go" . 2>/dev/null | grep -E "(route|handler|controller)" | head -20

# Verify that message queue operations occur through service boundaries, not direct client calls
grep -r "new.*Queue\|new.*Producer\|new.*Publisher" --include="*.js" --include="*.ts" --include="*.py" --include="*.java" --include="*.go" . 2>/dev/null | grep -v "service\|boundary\|adapter" | head -20

# Locate test suite configuration and verify service boundary implementations
find . -maxdepth 2 -type f \( -name 'jest.config.*' -o -name 'pytest.ini' -o -name 'karma.conf.*' -o -name 'mocha.opts' \) | head -1

# Search for mock or in-memory message queue implementations in test files
grep -r "mock.*[Qq]ueue\|in.*memory.*[Qq]ueue\|fake.*[Qq]ueue" --include="*.test.*" --include="*.spec.*" . 2>/dev/null | head -10
```

**Accept when:**
- All route handlers that invoke message queue operations do so through service boundaries that accept domain-level parameters and abstract messaging implementation details.
- Test suite execution confirms that service boundaries can be tested with mock implementations without requiring external messaging infrastructure.
- Error handling distinguishes between synchronous validation failures that block HTTP responses and asynchronous delivery failures that are logged with correlation identifiers.
- No direct imports of message queue client libraries exist in route handler modules; all such imports are confined to service boundary implementations.
- Service boundaries are implemented as dependency-injected interfaces with documented delivery semantics and failure modes.

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers invoking message queue operations MUST use service boundaries. Direct client library calls in route handlers are violations requiring refactoring.
</enforcement>