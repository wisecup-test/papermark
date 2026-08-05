# Isolate Asynchronous Message Operations Behind Dedicated Service Boundaries: Route Handlers Validate

These rules are ALWAYS ACTIVE for HTTP route handlers that invoke asynchronous message queue operations, service modules that encapsulate message queue client libraries, API endpoints that coordinate database persistence with message delivery, and error handling logic for both synchronous validation and asynchronous delivery failures.

### Rules

- **R-MSG-001** SHOULD: Route handlers should validate all input parameters before invoking message queue services to prevent invalid messages from entering the asynchronous processing pipeline.

### Verify

```bash
# Discover the project's dependency manifest and identify the build tool
# Inspect the lock or resolution artifact to determine exact versions of messaging client libraries
find . -name 'package.json' -o -name 'pom.xml' -o -name 'build.gradle' -o -name 'Gemfile' -o -name 'go.mod' | head -1

# Locate the project's test suite configuration and identify test scripts
find . -name 'jest.config.*' -o -name 'vitest.config.*' -o -name 'pytest.ini' -o -name '.mocharc.*' | head -1

# Search for route handler implementations that invoke message queue operations
grep -r "queue\|Queue\|enqueue\|Enqueue\|publish\|Publish" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" . | grep -E "(route|handler|endpoint)" | head -20

# Verify service boundaries are used instead of direct client library calls
grep -r "new.*Queue\|import.*queue.*client\|from.*queue.*client" --include="*.ts" --include="*.js" --include="*.py" --include="*.go" . | grep -v "service\|Service" | head -20

# Execute test suite to confirm service boundaries support mock implementations
grep -r "mock\|Mock\|stub\|Stub" --include="*.test.*" --include="*.spec.*" . | grep -i queue | head -10
```

**Accept when:**
- All route handlers that invoke message queue operations do so through service boundaries that accept domain-level parameters and abstract messaging implementation details.
- Test suite execution confirms that service boundaries can be tested with mock implementations without requiring external messaging infrastructure.
- Error handling distinguishes between synchronous validation failures that block HTTP responses and asynchronous delivery failures that are logged with correlation identifiers.
- No direct imports of message queue client libraries appear in route handler modules.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All route handlers invoking message queue operations MUST use service boundaries with input validation before message enqueuing.
</enforcement>