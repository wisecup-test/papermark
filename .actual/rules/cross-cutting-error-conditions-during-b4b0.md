# Adopt Asynchronous Message-Based AI Response Streaming via Service Abstraction: Error Conditions During

These rules are ALWAYS ACTIVE for all HTTP POST endpoints that handle AI chat message submission and streaming responses, service abstractions that coordinate message processing and AI invocation, database operations that validate chat access and persist generated titles, and document filtering services that resolve permitted document sets based on dataroom or link scope.

### Rules

- **R-STREAM-001** MUST: Error conditions during asynchronous streaming operations MUST be reported through both the stream controller error channel and console error logging to ensure visibility across transport and diagnostic boundaries.

### Verify

```bash
# Locate the project's test suite directory and identify integration tests for AI chat message endpoints
find . -type f -name '*test*' -o -name '*spec*' | grep -i chat | head -20

# Execute the test runner to verify that message submission produces streaming responses and handles error conditions correctly
# (Exact command depends on project's build tool — inspect lock file and manifest)

# Inspect the service abstraction module to confirm message processing function accepts all required context parameters
grep -r "def.*message.*stream\|function.*message.*stream" --include="*.py" --include="*.js" --include="*.ts" .

# Review error handling code paths to verify exceptions during streaming are reported through both channels
grep -r "stream.*error\|console.*error" --include="*.py" --include="*.js" --include="*.ts" . | grep -v node_modules | grep -v ".venv"

# Verify service abstraction interfaces accept required context parameters
grep -r "chat.*id\|vector.*store\|document.*filter" --include="*.py" --include="*.js" --include="*.ts" . | grep -E "(def|function|class)" | head -20
```

**Accept when:**
- Integration tests demonstrate that message submission endpoints successfully delegate to service abstractions and produce streaming responses with appropriate error handling.
- Service abstraction interfaces accept all required context parameters including chat identifier, message content, vector store identifier, and filtered document identifiers.
- Error conditions during streaming operations produce log entries in both stream controller error channels and console error output with sufficient context for incident investigation.
- Code review verification confirms new AI chat endpoints delegate message processing to service abstractions rather than implementing streaming logic inline.
- Integration test coverage requirements ensure that streaming error conditions are tested and produce expected dual logging output.
- Architecture review verification confirms service abstraction interfaces accept appropriate context parameters and maintain separation between transport and business logic concerns.

<enforcement>
Clause R-STREAM-001 verification is mandatory. Pull requests implementing inline streaming logic without service abstraction delegation must be rejected. Missing dual error logging paths must trigger automated test failures and block merge. Service interface changes omitting required context parameters must be flagged during code review.
</enforcement>