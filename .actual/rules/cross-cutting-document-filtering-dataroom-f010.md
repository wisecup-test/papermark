# Adopt Asynchronous Message-Based AI Response Streaming via Service Abstraction: Document Filtering Dataroom

These rules are ALWAYS ACTIVE for all HTTP POST endpoints that handle AI chat message submission and streaming responses, service abstractions that coordinate message processing and AI invocation, database operations that validate chat access and persist generated titles, and document filtering services that resolve permitted document sets based on dataroom or link scope.

### Rules

- **R-STREAM-001** SHOULD: Document filtering for dataroom-scoped chats SHOULD invoke a dedicated filtering service that resolves permitted document identifiers based on dataroom configuration and user permissions.

- **R-STREAM-002** MUST: Service abstraction boundaries MUST separate HTTP request handling concerns from message processing logic, enabling independent evolution of transport and business logic layers.

- **R-STREAM-003** MUST: Error handling during streaming operations MUST report exceptions through both stream controller error methods and console error logging with sufficient context for correlation.

- **R-STREAM-004** MUST: Database queries MUST combine access validation with configuration retrieval in a single operation, ensuring authorization checks complete before resource-intensive AI operations begin.

- **R-STREAM-005** SHOULD: Service abstraction invocations SHOULD accept all required context parameters including chat identifier, message content, vector store identifier, and filtered document identifiers without coupling to HTTP request structures.

- **R-STREAM-006** SHOULD: Title generation for new chat conversations SHOULD be invoked after successful message submission and persisted using a database update operation, handling failures gracefully without blocking message delivery.

### Verify

```bash
# Locate the project's test suite directory and identify integration tests for AI chat message endpoints
find . -type f -name "*test*" -o -name "*spec*" | grep -i chat | head -20

# Execute the test runner to verify message submission produces streaming responses
# (Exact command depends on project build tool — inspect lock file and manifest)

# Inspect the service abstraction module to confirm message processing function signature
grep -r "def.*message.*stream\|function.*message.*stream" --include="*.py" --include="*.js" --include="*.ts" .

# Review error handling code paths for dual logging
grep -r "stream.*error\|console.*error" --include="*.py" --include="*.js" --include="*.ts" . | grep -v node_modules | head -20

# Verify service abstraction accepts required context parameters
grep -r "chat.*id\|vector.*store\|filtered.*document" --include="*.py" --include="*.js" --include="*.ts" . | grep -v node_modules | head -20
```

**Accept when:**
- Integration tests demonstrate that message submission endpoints successfully delegate to service abstractions and produce streaming responses with appropriate error handling.
- Service abstraction interfaces accept all required context parameters including chat identifier, message content, vector store identifier, and filtered document identifiers.
- Error conditions during streaming operations produce log entries in both stream controller error channels and console error output with sufficient context for incident investigation.
- Database queries combine access validation with configuration retrieval in a single operation before AI operations begin.
- Title generation for new conversations is invoked after successful message submission and handles failures without blocking message delivery.

<enforcement>
Clause MUST NOT skip or defer verification. Code review verification that new AI chat endpoints delegate message processing to service abstractions rather than implementing streaming logic inline is mandatory. Integration test coverage requirements ensuring that streaming error conditions are tested and produce expected dual logging output are mandatory. Architecture review verification that service abstraction interfaces accept appropriate context parameters and maintain separation between transport and business logic concerns is mandatory.
</enforcement>