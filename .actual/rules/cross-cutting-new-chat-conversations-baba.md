# Adopt Asynchronous Message-Based AI Response Streaming via Service Abstraction: New Chat Conversations

These rules are ALWAYS ACTIVE for all HTTP POST endpoints that handle AI chat message submission and streaming responses, service abstractions that coordinate message processing and AI invocation, database operations that validate chat access and persist generated titles, and document filtering services that resolve permitted document sets based on dataroom or link scope.

### Rules

- **R-CHAT-001** SHOULD: New chat conversations with no existing messages SHOULD trigger automatic title generation using a dedicated title generation service, with the generated title persisted via database update.

### Verify

```bash
# Locate the project's test suite directory and identify integration tests for AI chat message endpoints
find . -type f -name '*test*' -o -name '*spec*' | grep -i chat | head -20

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
- New chat conversations trigger title generation service invocation after successful message submission.
- Generated titles are persisted via database update operation with graceful failure handling that does not block message delivery.

<enforcement>
Clause Code MUST NOT skip or defer verification. All new chat conversation endpoints MUST delegate to service abstractions. All streaming error conditions MUST produce dual logging output. Title generation MUST be invoked for new conversations with no existing messages.
</enforcement>