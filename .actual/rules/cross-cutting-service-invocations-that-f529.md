# Adopt Asynchronous Message-Based AI Response Streaming via Service Abstraction: Service Invocations That

These rules are ALWAYS ACTIVE for all HTTP POST endpoints that handle AI chat message submission and streaming responses, service abstractions that coordinate message processing and AI invocation, database operations that validate chat access and persist generated titles, and document filtering services that resolve permitted document sets based on dataroom or link scope.

### Rules

- **R-STREAM-001** MUST: Service invocations that produce streaming responses MUST accept all required context parameters including chat identifier, message content, vector store identifier, filtered document identifiers, and optional dataroom or link identifiers.

### Verify

```bash
# Locate the project's test suite directory and identify integration tests for AI chat message endpoints
find . -type f -name "*test*" -o -name "*spec*" | grep -i chat | head -20

# Execute the test runner to verify that message submission produces streaming responses
# (Exact command depends on project's build tool — inspect lock file and manifest)

# Inspect the service abstraction module to confirm message processing function signature
grep -r "def.*message.*stream\|function.*message.*stream" --include="*.py" --include="*.js" --include="*.ts" .

# Verify error handling captures exceptions and reports through both stream controller and console
grep -r "stream.*error\|console.*error" --include="*.py" --include="*.js" --include="*.ts" . | grep -i "chat\|message"

# Review database queries combining access validation with configuration retrieval
grep -r "SELECT.*WHERE.*access\|query.*permission" --include="*.py" --include="*.js" --include="*.sql" .
```

**Accept when:**
- Integration tests demonstrate that message submission endpoints successfully delegate to service abstractions and produce streaming responses with appropriate error handling.
- Service abstraction interfaces accept all required context parameters including chat identifier, message content, vector store identifier, and filtered document identifiers.
- Error conditions during streaming operations produce log entries in both stream controller error channels and console error output with sufficient context for incident investigation.
- Database queries combine access validation with configuration retrieval in a single operation, ensuring authorization checks complete before resource-intensive AI operations begin.
- New chat conversations invoke title generation service after successful message submission and persist the generated title using database update operations.

<enforcement>
Clause R-STREAM-001 is mandatory. Code review MUST verify that new AI chat endpoints delegate message processing to service abstractions rather than implementing streaming logic inline. Integration test coverage MUST ensure streaming error conditions are tested and produce expected dual logging output. Architecture review MUST verify that service abstraction interfaces accept appropriate context parameters and maintain separation between transport and business logic concerns. Pull requests implementing inline streaming logic without service abstraction delegation are rejected. Missing error handling for streaming operations triggers automated test failures and blocks merge. Service interface changes omitting required context parameters are flagged during code review and require revision.
</enforcement>