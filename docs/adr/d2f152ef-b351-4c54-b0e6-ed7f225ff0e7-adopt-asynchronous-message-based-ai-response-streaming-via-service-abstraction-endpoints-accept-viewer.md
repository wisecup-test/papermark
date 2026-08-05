# Adopt Asynchronous Message-Based AI Response Streaming via Service Abstraction: Endpoints Accept Viewer

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The API endpoint handles AI chat message submission and response streaming, requiring coordination between HTTP request handling, database persistence, and AI service invocation.
- Chat operations require access control validation, document filtering based on dataroom permissions, and automatic title generation for new conversations.
- The streaming response pattern necessitates asynchronous error handling with dual logging paths: stream controller errors and console error reporting.
- Integration with vector stores and dataroom document filtering requires passing multiple context parameters through service boundaries.

## Problem Statement

API endpoints that orchestrate AI chat operations must coordinate HTTP request handling, database transactions, permission validation, document filtering, and streaming AI responses while maintaining clear separation between transport concerns and business logic, ensuring error visibility across asynchronous boundaries.

## Decision

1. MAY: API endpoints MAY accept viewer identifier as a query parameter to support cache layer differentiation or audit logging requirements.

## Policy Block

- MAY API endpoints MAY accept viewer identifier as a query parameter to support cache layer differentiation or audit logging requirements.

In scope:
- HTTP POST endpoints that handle AI chat message submission and streaming responses
- Service abstractions that coordinate message processing, AI invocation, and response streaming
- Database operations that validate chat access, retrieve configuration, and persist generated titles
- Document filtering services that resolve permitted document sets based on dataroom or link scope

Out of scope:
- Internal implementation details of AI model invocation or prompt construction
- Vector store indexing or embedding generation processes
- WebSocket or alternative streaming transport mechanisms
- Client-side message rendering or UI state management

## Rationale

- The evidence shows explicit separation between route handler concerns and message processing logic through service abstraction invocation, enabling independent evolution of transport and business logic layers.
- Dual error reporting through both stream controller and console logging addresses the challenge of error visibility in asynchronous streaming contexts where transport-level errors may not surface through standard HTTP error responses.
- The pattern of passing multiple context parameters to the service abstraction reflects the need to coordinate vector store access, document filtering, and dataroom/link scoping without coupling the service to HTTP request structures.
- Database queries that combine access validation with configuration retrieval optimize round-trips while ensuring authorization checks precede resource-intensive AI operations.

## Consequences

Positive:
- Clear separation of concerns enables independent testing of route handling, access validation, and message processing logic.
- Service abstraction boundary allows AI streaming implementation to evolve without modifying HTTP endpoint code.
- Dual error logging ensures diagnostic visibility even when stream transport errors prevent standard HTTP error responses.
- Parameterized service invocation supports multiple chat scoping models including dataroom-based and link-based filtering without service interface changes.

Negative:
- Multiple service invocations per request increase latency and introduce additional failure points requiring coordinated error handling.
- Passing numerous context parameters through service boundaries creates coupling between route handler data extraction logic and service interface expectations.
- Dual error logging may produce redundant or inconsistent error messages requiring correlation during incident investigation.
- Automatic title generation for new chats introduces additional database write operations that may fail independently of message processing.

## Alternatives

- Implement AI streaming logic directly within route handler using inline model invocation and response construction (rejected)
  Rejected because: Inline implementation couples HTTP transport concerns with AI model interaction, vector store access, and document filtering logic, preventing independent testing and evolution of streaming behavior.
  When valid: Valid for prototype or proof-of-concept implementations where service abstraction overhead outweighs maintainability benefits.
- Use message queue or event bus to decouple message submission from response generation, returning request identifier immediately (rejected)
  Rejected because: Asynchronous decoupling breaks the streaming response contract expected by clients, requiring alternative mechanisms for response delivery such as webhooks or polling.
  When valid: Valid for batch processing scenarios or when client requirements permit asynchronous response delivery with separate notification channels.
- Consolidate access validation, document filtering, and title generation into a single orchestration service invocation (deferred)
  Rejected because: Consolidation may improve performance by reducing service boundary crossings but requires careful design to maintain separation of concerns and testability.
  When valid: Valid when profiling demonstrates that multiple service invocations create unacceptable latency or when orchestration complexity justifies a dedicated coordination layer.

## Risks

- Service abstraction failure after successful access validation may leave chat in inconsistent state if title generation or message persistence fails independently of streaming response delivery.
  Mitigation: Implement transactional boundaries or compensating operations to ensure chat state consistency, and monitor for partial failure patterns indicating coordination issues.
  Owner: engineering team
- Dual error logging may produce inconsistent or incomplete error information if stream controller errors and console errors capture different exception details or stack traces.
  Mitigation: Standardize error serialization and ensure both logging paths capture complete error context including request identifiers for correlation during incident investigation.
  Owner: engineering team
- Parameter proliferation in service abstraction interface may create maintenance burden as new filtering or scoping requirements emerge, requiring coordinated changes across route handlers and service implementations.
  Mitigation: Consider introducing a context object or parameter object pattern to encapsulate related parameters and reduce interface coupling as requirements evolve.
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Locate the service abstraction module that handles message submission and streaming response generation. Ensure all required context parameters are extracted from the validated request body and query parameters before invocation.
- Implement error handling that captures exceptions during streaming operations and reports them through both the stream controller error method and console error logging, including sufficient context for correlation.
- Structure database queries to combine access validation with configuration retrieval in a single operation, ensuring authorization checks complete before resource-intensive AI operations begin.
- For new chat conversations, invoke the title generation service after successful message submission and persist the generated title using a database update operation, handling title generation failures gracefully without blocking message delivery.

## Continuation Context


Verify commands:
- Locate the project's test suite directory and identify integration tests for AI chat message endpoints. Execute the test runner to verify that message submission produces streaming responses and handles error conditions correctly.
- Inspect the service abstraction module to confirm that the message processing function accepts all required context parameters and returns a streaming response object compatible with the HTTP framework's streaming API.
- Review error handling code paths to verify that exceptions during streaming operations are reported through both stream controller error channels and console logging with consistent error context.

Accept when:
- Integration tests demonstrate that message submission endpoints successfully delegate to service abstractions and produce streaming responses with appropriate error handling.
- Service abstraction interfaces accept all required context parameters including chat identifier, message content, vector store identifier, and filtered document identifiers.
- Error conditions during streaming operations produce log entries in both stream controller error channels and console error output with sufficient context for incident investigation.

## Enforcement

- Verified by: Code review verification that new AI chat endpoints delegate message processing to service abstractions rather than implementing streaming logic inline.
- Verified by: Integration test coverage requirements ensuring that streaming error conditions are tested and produce expected dual logging output.
- Verified by: Architecture review verification that service abstraction interfaces accept appropriate context parameters and maintain separation between transport and business logic concerns.
- Violation handling: Pull requests that implement inline streaming logic within route handlers without service abstraction delegation are rejected with guidance to refactor using the established pattern.
- Violation handling: Missing error handling for streaming operations triggers automated test failures and blocks merge until dual logging paths are implemented.
- Violation handling: Service interface changes that omit required context parameters are flagged during code review and require revision to maintain consistency with established integration patterns.
- Exception process: Exception requests must document specific technical constraints that prevent service abstraction usage, such as performance requirements or framework limitations.
- Exception process: Architecture review team evaluates exception requests and may approve alternative patterns that maintain equivalent separation of concerns.
- Exception process: Approved exceptions are documented with rationale and expiration criteria to ensure periodic reevaluation as constraints evolve.