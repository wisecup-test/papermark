# Parse External JSON Payloads Before Use in Archive Workflows: Validation Failures Result

These rules are ALWAYS ACTIVE for all code paths that receive external JSON payloads from Lambda invocation results, trigger events, decoded payload strings, or any external data structure used in database queries, storage operations, or workflow coordination within archive workflows.

### Rules

- **R-PARSE-001** MUST: Parse all external JSON payloads using explicit JSON.parse() calls immediately after receiving them and before any property access or business logic execution.
- **R-PARSE-002** MUST: Wrap all JSON.parse() calls in try-catch blocks with error logging that includes payload source, expected schema version, and sufficient context for debugging without exposing sensitive payload content.
- **R-PARSE-003** SHOULD: Validation failures SHOULD result in early termination of the workflow with appropriate error responses rather than attempting to proceed with partial or default data.
- **R-PARSE-004** MUST: Prevent any direct property access on unparsed external payload variables; all property access must occur only after successful parsing.
- **R-PARSE-005** MUST: Ensure parse operations include error handling that prevents malformed data from reaching database or storage operations.

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
- All JSON.parse() calls are wrapped in try-catch blocks with structured error logging
- Validation failures result in early workflow termination with appropriate error responses

<enforcement>
Claude Code MUST NOT skip or defer verification. All external payload parsing must be verified before code is committed. Static analysis and integration tests must pass to confirm compliance with R-PARSE-001 through R-PARSE-005.
</enforcement>