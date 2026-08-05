# Parse External JSON Payloads Before Use in Archive Workflows: Parsed Payload Structures

These rules are ALWAYS ACTIVE for all code paths that receive external JSON payloads from Lambda invocations, trigger events, decoded payload strings, or any external data structure used in database operations, storage uploads, or workflow coordination logic.

### Rules

- **R-PAYLOAD-001** MUST: Parsed payload structures MUST be validated against expected schemas before use in database operations, storage uploads, or workflow coordination logic.
- **R-PAYLOAD-002** MUST: Position JSON parsing immediately after receiving external payloads and before any property access or business logic execution to establish the earliest possible validation boundary.
- **R-PAYLOAD-003** MUST: Wrap all JSON.parse calls in try-catch blocks with error logging and graceful degradation paths to prevent synchronous exceptions from crashing the workflow.
- **R-PAYLOAD-004** SHOULD: Implement structured error logging for parse failures that includes payload source, expected schema version, and sufficient context for debugging without exposing sensitive payload content.
- **R-PAYLOAD-005** SHOULD: Extract validation logic into reusable functions or middleware that can be consistently applied across all external integration points in the archive workflow.

### Verify

```bash
# Discover the project's test runner from the dependency manifest and execute the test suite covering archive workflow payload handling
# (Exact command depends on build tool discovered from manifest)

# Locate the project's static analysis configuration and run type checking to verify payload parsing occurs before property access
# (Exact command depends on static analysis tool configured in project)

# Identify the project's linting configuration and verify no direct property access on unparsed external payloads is flagged
# (Exact command depends on linter configured in project)
```

**Accept when:**
- All external JSON payloads from Lambda results and trigger events are explicitly parsed before use
- Parse operations include error handling that prevents malformed data from reaching database or storage operations
- Static analysis confirms no direct property access on unparsed external payload variables
- Structured error logging is present for parse failures with payload source and schema context
- Validation logic is extracted into reusable functions or middleware applied consistently across integration points

<enforcement>
Clause Code MUST NOT skip or defer verification. All external payloads must be explicitly parsed at ingress points with proper error handling before any business logic execution or property access occurs.
</enforcement>