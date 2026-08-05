# Define Service Boundaries Through Explicit Task Definitions: Task Definitions Export

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase implements long-running background operations that require coordination across multiple subsystems including storage, database persistence, event streaming, and external service invocation
- Operations involve multi-step workflows with progress tracking, error handling, and state management that span beyond single request-response cycles
- The system uses @trigger.dev/sdk to define discrete service boundaries for asynchronous task execution, separating orchestration logic from business logic
- Task definitions export typed payload contracts and task identifiers, establishing explicit interfaces between service boundaries
- Evidence shows structured logging, metadata caching, event-driven coordination, and AWS SDK integration within task boundaries

## Problem Statement

Background operations that coordinate multiple subsystems require clear service boundaries to manage complexity, enable independent testing, and support distributed execution. Without explicit task definitions and payload contracts, these operations become tightly coupled to request handlers, making them difficult to scale, monitor, and maintain across deployment boundaries.

## Decision

1. MUST: Task definitions MUST export both the payload type contract and the task identifier to establish service boundaries

## Policy Block

- MUST Task definitions MUST export both the payload type contract and the task identifier to establish service boundaries

In scope:
- Background operations requiring multi-step coordination
- Asynchronous workflows with progress tracking requirements
- Operations that span multiple subsystem boundaries
- Long-running tasks requiring independent scaling and monitoring

Out of scope:
- Synchronous request-response handlers
- Single-subsystem operations without coordination requirements
- Inline utility functions within service implementations
- Pure computation tasks without external dependencies

## Rationale

- The evidence shows explicit task definition exports with typed payload contracts, establishing clear service boundaries for the dataroom freeze archive operation
- Structured logging with contextual metadata appears consistently throughout the task implementation, enabling distributed tracing and observability
- The pattern uses event-driven coordination through archive entry handlers and error handlers, decoupling subsystem interactions
- Integration with AWS SDK clients and external service invocation demonstrates the need for well-defined boundaries to manage distributed system complexity

## Consequences

Positive:
- Clear service boundaries enable independent testing, deployment, and scaling of background operations
- Explicit payload contracts provide type safety and documentation at service boundaries
- Structured logging and metadata caching enable comprehensive observability across distributed task execution
- Event-driven coordination reduces coupling between subsystems and improves maintainability

Negative:
- Additional abstraction layer increases initial development complexity for simple operations
- Task definition boilerplate and payload contract maintenance add overhead to codebase
- Distributed execution introduces latency and complexity in error handling compared to inline operations
- Debugging across service boundaries requires correlation of logs and events across multiple execution contexts

## Alternatives

- Implement background operations as inline functions within request handlers (rejected)
  Rejected because: Inline implementations couple orchestration to request lifecycle, preventing independent scaling and making long-running operations block request threads
  When valid: Only for trivial operations completing within milliseconds with no coordination requirements
- Use generic message queue with untyped payloads (rejected)
  Rejected because: Untyped payloads eliminate compile-time safety and make service contracts implicit rather than explicit, increasing maintenance burden
  When valid: When integrating with legacy systems that cannot support typed contracts
- Implement custom task orchestration framework (rejected)
  Rejected because: Building custom orchestration duplicates functionality available in existing task execution frameworks and increases maintenance surface
  When valid: When existing frameworks cannot meet specific performance or feature requirements

## Risks

- Task execution framework dependency creates vendor lock-in and migration complexity
  Mitigation: Isolate framework-specific code to task definition layer; keep business logic framework-agnostic
  Owner: engineering team
- Distributed task execution introduces failure modes not present in synchronous operations
  Mitigation: Implement comprehensive error handling, retry logic, and dead-letter queues; use structured logging for failure diagnosis
  Owner: engineering team
- Payload contract evolution may break compatibility with in-flight tasks
  Mitigation: Version payload contracts explicitly; maintain backward compatibility during transitions; implement payload validation
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
- Task definitions should be colocated with their domain logic but exported through a public contract module to establish clear boundaries
- Progress tracking through metadata caching should use normalized progress values between 0 and 1 with human-readable status text
- Event handlers for subsystem coordination should implement proper error propagation and cleanup to prevent resource leaks

## Continuation Context


Verify commands:
- Discover the project's task execution framework configuration and locate the task registry or definition index
- Identify the project's testing framework and execute tests that validate task payload contracts and execution boundaries
- Locate the project's logging configuration and verify structured logging output includes required contextual metadata fields

Accept when:
- All background operations with multi-subsystem coordination are defined as discrete tasks with exported payload contracts
- Task implementations include structured logging with contextual metadata at key execution points
- Tests validate task execution boundaries and payload contract compliance

## Enforcement

- Verified by: Code review verification that background operations follow task definition pattern
- Verified by: Static analysis verification that task definitions export required payload contracts
- Verified by: Integration tests validating task execution and observability requirements
- Violation handling: Pull requests introducing background operations without task definitions are rejected
- Violation handling: Missing payload contracts or structured logging trigger automated review comments
- Violation handling: Violations identified in production code are tracked as technical debt items with remediation timeline
- Exception process: Exception requests must document why task definition pattern is inappropriate for the specific operation
- Exception process: Architecture review board evaluates exception requests against scope criteria
- Exception process: Approved exceptions are documented with rationale and reviewed quarterly for continued validity