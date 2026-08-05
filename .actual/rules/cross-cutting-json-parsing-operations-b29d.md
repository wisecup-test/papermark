# Parse External JSON Payloads Before Use in Archive Workflows: Json Parsing Operations

These rules are ALWAYS ACTIVE for all code paths that receive external JSON payloads from Lambda invocation results, trigger events, decoded payload strings, or any data structure originating from outside the application boundary before use in archive workflows, database queries, storage operations, or workflow coordination.

### Rules

- **R-JSON-001** MUST: JSON parsing operations MUST include error handling that logs parse failures with sufficient context for debugging and prevents malformed data from propagating to downstream systems.
- **R-JSON-002** MUST: Position JSON parsing immediately after receiving external payloads and before any property access or business logic execution to establish the earliest possible validation boundary.
- **R-JSON-003** MUST: Implement structured error logging for parse failures that includes payload source, expected schema version, and sufficient context for debugging without exposing sensitive payload content.
- **R-JSON-004** SHOULD: Consider extracting validation logic into reusable functions or middleware that can be consistently applied across all external integration points in the archive workflow.

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
- Structured error logging is present for all JSON.parse operations with payload source and schema context
- Parse operations are positioned immediately after payload receipt, before any business logic execution

<enforcement>
Clause Code MUST NOT skip or defer verification. All external payloads MUST be explicitly parsed at ingress points with comprehensive error handling before propagation to downstream systems. Violations block code review and continuous integration deployment.
</enforcement>