# Adopt Asynchronous Message-Based AI Response Streaming via Service Abstraction: Request Body Validation

These rules are ALWAYS ACTIVE for all HTTP POST endpoints that handle AI chat message submission, service abstractions that coordinate message processing and AI invocation, database operations that validate chat access and persist generated titles, and document filtering services that resolve permitted document sets based on dataroom or link scope.

### Rules

- **R-STREAM-001** MUST: Request body validation MUST use schema-based parsing with safe parse semantics, returning structured validation errors when input does not conform to expected message schema.

### Verify

```bash
# Locate the project's test suite directory and identify integration tests for AI chat message endpoints
find . -type f -name '*test*' -o -name '*spec*' | grep -i chat | head -20

# Execute the test runner to verify that message submission produces streaming responses and handles error conditions correctly
# (Exact command depends on project's build tool — inspect package manifest and lock file)

# Inspect the service abstraction module to confirm message processing function accepts all required context parameters
grep -r "message.*processing\|streaming.*response" --include="*.ts" --include="*.js" | grep -i service | head -10

# Review error handling code paths to verify exceptions during streaming are reported through both stream controller and console logging
grep -r "stream.*error\|console.*error" --include="*.ts" --include="*.js" | grep -i chat | head -10
```

**Accept when:**
- Integration tests demonstrate that message submission endpoints successfully delegate to service abstractions and produce streaming responses with appropriate error handling.
- Service abstraction interfaces accept all required context parameters including chat identifier, message content, vector store identifier, and filtered document identifiers.
- Error conditions during streaming operations produce log entries in both stream controller error channels and console error output with sufficient context for incident investigation.
- Request body validation uses schema-based parsing (e.g., Zod, Joi, or equivalent) with safe parse semantics that return structured validation errors rather than throwing exceptions.

<enforcement>
Clause Code MUST NOT skip or defer verification. All new AI chat endpoints MUST delegate message processing to service abstractions with schema-based request validation. Pull requests implementing inline streaming logic without service abstraction are rejected. Missing dual error logging blocks merge. Service interface changes omitting required context parameters are flagged during code review.
</enforcement>