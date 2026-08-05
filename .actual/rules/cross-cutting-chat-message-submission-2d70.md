# Adopt Asynchronous Message-Based AI Response Streaming via Service Abstraction: Chat Message Submission

These rules are ALWAYS ACTIVE for all HTTP POST endpoints that handle AI chat message submission and streaming responses, service abstractions that coordinate message processing and AI invocation, database operations that validate chat access and persist generated titles, and document filtering services that resolve permitted document sets based on dataroom or link scope.

### Rules

- **R-CHAT-001** MUST: AI chat message submission endpoints MUST delegate message processing and response generation to a dedicated service abstraction rather than implementing streaming logic inline within the route handler.

### Verify

```bash
# Locate the project's test suite directory and identify integration tests for AI chat message endpoints
find . -type f -name "*test*" -o -name "*spec*" | grep -i chat | head -20

# Execute the test runner to verify that message submission produces streaming responses and handles error conditions correctly
# (Exact command depends on project's build tool — inspect lock file and manifest)

# Inspect the service abstraction module to confirm message processing function accepts all required context parameters
grep -r "def.*message.*submit\|function.*message.*submit" --include="*.py" --include="*.js" --include="*.ts" | head -10

# Review error handling code paths to verify exceptions during streaming are reported through both stream controller and console logging
grep -r "stream.*error\|console.*error" --include="*.py" --include="*.js" --include="*.ts" | grep -i chat | head -10
```

**Accept when:**
- Integration tests demonstrate that message submission endpoints successfully delegate to service abstractions and produce streaming responses with appropriate error handling.
- Service abstraction interfaces accept all required context parameters including chat identifier, message content, vector store identifier, and filtered document identifiers.
- Error conditions during streaming operations produce log entries in both stream controller error channels and console error output with sufficient context for incident investigation.
- Route handlers do not contain inline AI model invocation, prompt construction, or response streaming logic.

<enforcement>
Clause Code MUST NOT skip or defer verification. Pull requests implementing inline streaming logic without service abstraction delegation are rejected. Missing dual error logging paths block merge. Service interface changes omitting required context parameters are flagged during code review.
</enforcement>