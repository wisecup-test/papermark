# Isolate Asynchronous Message Operations Behind Dedicated Service Boundaries: Message Queue Service

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- HTTP route handlers in the codebase coordinate synchronous request processing with asynchronous message operations, requiring clear boundaries between immediate response logic and deferred work.
- Message queue operations are invoked from API endpoints that also perform database queries, input validation, and error handling, creating coupling between transport, persistence, and messaging concerns.
- The pattern emerges in routes handling AI chat messages and scheduled user onboarding workflows, where message delivery must be decoupled from HTTP response cycles.
- Console-based error logging is present alongside message queue invocations, indicating observability requirements for asynchronous operation failures that occur outside the request context.

## Problem Statement

API route handlers that directly invoke message queue operations without architectural boundaries create tight coupling between HTTP lifecycle management, persistence logic, and asynchronous messaging concerns. This coupling complicates error handling, observability, testing, and independent evolution of messaging infrastructure, while obscuring the distinction between synchronous request processing and deferred asynchronous work.

## Decision

1. MAY: Message queue service implementations may support multiple transport backends through adapter patterns to enable testing with in-memory queues and production deployment with distributed messaging systems.

## Policy Block

- MAY Message queue service implementations may support multiple transport backends through adapter patterns to enable testing with in-memory queues and production deployment with distributed messaging systems.

In scope:
- HTTP route handlers that invoke asynchronous message operations
- Service modules that encapsulate message queue client libraries
- API endpoints that coordinate database persistence with message delivery
- Error handling logic for both synchronous validation and asynchronous delivery failures

Out of scope:
- Synchronous HTTP endpoints that do not involve message queue operations
- Message queue consumer implementations that process messages from queues
- Database transaction boundaries that do not coordinate with messaging
- Client-side code that invokes HTTP endpoints

## Rationale

- The evidence shows message queue operations invoked directly from route handlers alongside database queries and error logging, indicating coupling between transport, persistence, and messaging layers that complicates independent evolution.
- Encapsulating message operations behind service boundaries enables testing with mock implementations, supports migration between messaging technologies, and clarifies the architectural distinction between synchronous request processing and asynchronous work.
- Separating error handling for synchronous validation from asynchronous delivery failures improves observability by making it explicit which failures block HTTP responses and which require separate monitoring and retry mechanisms.
- The pattern appears in multiple route handlers with similar structure, suggesting that standardizing the service boundary approach will reduce duplication and establish consistent error handling and observability practices across asynchronous operations.

## Consequences

Positive:
- Service boundaries decouple HTTP route handlers from message queue implementation details, enabling independent testing, migration between messaging technologies, and evolution of transport protocols.
- Explicit separation of synchronous validation from asynchronous delivery improves error handling clarity and enables appropriate observability mechanisms for each failure mode.
- Standardized service interfaces for message operations reduce code duplication across route handlers and establish consistent patterns for coordinating persistence with asynchronous work.
- Abstraction of queue-specific configuration behind service boundaries simplifies route handler logic and consolidates messaging infrastructure concerns in dedicated modules.

Negative:
- Introducing service boundaries adds indirection that may obscure the direct relationship between HTTP requests and message queue operations during initial code comprehension.
- Additional abstraction layers increase the number of modules and interfaces that must be maintained, potentially complicating dependency management and build configuration.
- Service boundaries may introduce performance overhead if not designed carefully, particularly if they add unnecessary serialization, validation, or context-switching between layers.
- Standardizing service interfaces across diverse message queue use cases may require compromise between generality and specificity, potentially leading to either overly generic or overly specialized abstractions.

## Alternatives

- Invoke message queue client libraries directly from route handlers without service boundaries (rejected)
  Rejected because: Direct invocation couples HTTP handlers to specific messaging implementations, complicates testing with mock queues, and prevents independent evolution of messaging infrastructure without modifying route handler code.
  When valid: In prototypes or single-use scripts where messaging infrastructure will not change and testing with mock implementations is not required.
- Use a generic event bus abstraction that handles all asynchronous operations including message queues, background jobs, and webhooks (rejected)
  Rejected because: A unified event bus abstraction may obscure important semantic differences between message queue operations, scheduled jobs, and webhook delivery, leading to inappropriate retry policies, ordering guarantees, or delivery semantics.
  When valid: In systems where all asynchronous operations share identical delivery guarantees, retry policies, and observability requirements, making a unified abstraction appropriate.
- Implement message queue operations as middleware that intercepts HTTP responses and enqueues messages based on response metadata (rejected)
  Rejected because: Middleware-based message enqueuing couples message content to HTTP response structure, complicates error handling when message delivery fails after the response is sent, and obscures the explicit relationship between business logic and asynchronous operations.
  When valid: In systems where message queue operations are purely side effects of successful HTTP responses and do not require coordination with request-specific business logic or database transactions.

## Risks

- Service boundaries may be implemented inconsistently across different route handlers, leading to fragmented patterns and reduced maintainability.
  Mitigation: Establish a reference implementation and code review guidelines that enforce consistent service boundary patterns. Document the standard interface contract and provide examples for common use cases.
  Owner: engineering team
- Abstraction of message queue operations may hide important failure modes or delivery guarantees that affect application correctness.
  Mitigation: Document the delivery semantics, retry policies, and failure modes of the service boundary interface. Ensure that service implementations expose sufficient observability to diagnose asynchronous delivery failures.
  Owner: engineering team
- Service boundaries that coordinate database transactions with message delivery may introduce distributed transaction complexity or inconsistency between persisted state and enqueued messages.
  Mitigation: Adopt transactional outbox pattern or similar techniques to ensure consistency between database writes and message delivery. Document the consistency guarantees provided by the service boundary.
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
- Service boundaries should accept domain-level parameters that describe the message content and routing requirements without exposing queue names, connection strings, or protocol-specific configuration. This enables route handlers to remain agnostic to messaging infrastructure changes.
- Error handling should distinguish between validation failures that occur before message enqueuing and delivery failures that occur asynchronously. Validation failures should block the HTTP response with appropriate error codes, while delivery failures should be logged with correlation identifiers for debugging.
- Consider implementing service boundaries as dependency-injected interfaces to enable testing with in-memory implementations and production deployment with distributed queue clients. This approach simplifies unit testing and integration testing without requiring external messaging infrastructure.

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify the build tool. Inspect the lock or resolution artifact to determine the exact versions of all messaging client libraries.
- Locate the project's test suite configuration and identify test scripts that verify service boundary implementations. Execute the test suite to confirm that message queue operations are properly encapsulated behind service interfaces.
- Search the codebase for route handler implementations that invoke message queue operations. Verify that all such invocations occur through service boundaries rather than direct client library calls.

Accept when:
- All route handlers that invoke message queue operations do so through service boundaries that accept domain-level parameters and abstract messaging implementation details.
- Test suite execution confirms that service boundaries can be tested with mock implementations without requiring external messaging infrastructure.
- Error handling distinguishes between synchronous validation failures that block HTTP responses and asynchronous delivery failures that are logged with correlation identifiers.

## Enforcement

- Verified by: Code review process verifies that new route handlers invoking message queue operations use service boundaries rather than direct client library calls.
- Verified by: Automated static analysis or linting rules detect direct imports of message queue client libraries in route handler modules.
- Verified by: Integration tests verify that service boundaries properly abstract messaging implementation details and support testing with mock implementations.
- Violation handling: Code review feedback requires refactoring of route handlers that directly invoke message queue client libraries to use service boundaries instead.
- Violation handling: Pull requests that introduce direct message queue client library calls in route handlers are blocked until service boundaries are implemented.
- Violation handling: Existing violations are tracked as technical debt items and prioritized for refactoring based on the frequency of changes to affected route handlers.
- Exception process: Exceptions may be granted for prototype code or proof-of-concept implementations that will not be deployed to production environments.
- Exception process: Exception requests must document the specific technical constraint that prevents service boundary implementation and the plan for eventual compliance.
- Exception process: Approved exceptions are time-limited and require re-evaluation before the code is promoted to production deployment.