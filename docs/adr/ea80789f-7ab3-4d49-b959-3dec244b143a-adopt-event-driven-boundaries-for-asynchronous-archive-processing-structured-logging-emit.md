# Adopt Event-Driven Boundaries for Asynchronous Archive Processing: Structured Logging Emit

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The dataroom freeze archive feature processes large document collections asynchronously, requiring coordination between archive creation, cloud storage upload, and progress tracking without blocking the main request thread
- Archive generation involves streaming transformations where chunks flow through hash computation, compression, and upload stages, necessitating event-based coordination to handle errors and completion across multiple concurrent streams
- The system integrates with cloud object storage and serverless compute services, requiring event-driven invocation patterns to trigger downstream processing and handle asynchronous responses
- Progress metadata updates and batch processing coordination require decoupled communication between the archive generation pipeline and state management layer to maintain responsiveness during long-running operations

## Problem Statement

Long-running archive generation operations that combine streaming compression, cryptographic hashing, multipart cloud uploads, and database state updates require a coordination mechanism that prevents blocking, enables error propagation across pipeline stages, supports progress tracking, and allows independent scaling of compute-intensive tasks without introducing tight coupling between components.

## Decision

1. SHOULD: Structured logging SHOULD emit events at pipeline stage boundaries including batch creation, completion, and document collection milestones

## Policy Block

- SHOULD Structured logging SHOULD emit events at pipeline stage boundaries including batch creation, completion, and document collection milestones

In scope:
- Asynchronous archive generation workflows involving streaming compression and cloud uploads
- Long-running document processing tasks that require progress tracking and cancellation support
- Pipeline stages that coordinate multiple concurrent streams with interdependent error handling
- Serverless function invocations that trigger downstream processing asynchronously

Out of scope:
- Synchronous request-response API endpoints that complete within typical HTTP timeout windows
- Database transaction boundaries that require ACID guarantees without asynchronous coordination
- Simple file I/O operations that do not involve streaming transformations or cloud service integration
- Batch jobs that process items sequentially without concurrent stream coordination

## Rationale

- The evidence shows explicit event handler registration on archive objects using on('entry'), on('error') patterns, demonstrating event-driven coordination between archive generation and downstream processing stages
- Error propagation through multiple stream layers with abort and destroy operations indicates the need for event-based error handling to maintain pipeline integrity across asynchronous boundaries
- Cloud service client send operations and serverless function invocations use promise-based asynchronous patterns that naturally align with event-driven architectures for non-blocking execution
- Progress metadata updates through cache layer set operations and structured logging at stage boundaries show decoupled state management enabled by event-driven communication patterns

## Consequences

Positive:
- Non-blocking execution allows the system to handle multiple concurrent archive generation requests without thread pool exhaustion or request timeout failures
- Event-based error propagation ensures that failures in any pipeline stage trigger coordinated cleanup across all dependent streams and cloud resources
- Decoupled progress tracking through cache layer events enables real-time status updates without blocking the main processing pipeline
- Stream-based event handlers maintain natural backpressure through the pipeline, preventing memory exhaustion when processing large document collections

Negative:
- Event-driven control flow increases debugging complexity as execution paths span multiple asynchronous callbacks and promise chains rather than linear synchronous code
- Error handling requires explicit propagation logic across event boundaries, increasing the risk of unhandled promise rejections or orphaned resources if handlers are incomplete
- Testing event-driven pipelines requires mock event emitters and careful sequencing of asynchronous assertions, increasing test complexity and flakiness potential
- Performance profiling and tracing becomes more difficult as execution spans multiple event loop ticks and callback queues rather than contiguous stack frames

## Alternatives

- Synchronous blocking pipeline with sequential processing of all archive stages in a single request handler (rejected)
  Rejected because: Blocking operations would exceed typical HTTP timeout windows for large document collections and prevent concurrent request handling, causing user-facing failures and poor resource utilization
  When valid: Only viable for small archives with guaranteed sub-second processing times and systems with no concurrency requirements
- Message queue-based coordination with explicit queue publish and subscribe operations for each pipeline stage (rejected)
  Rejected because: Introduces additional infrastructure dependencies and operational complexity for coordination that can be achieved with native stream events within a single process boundary
  When valid: Appropriate when pipeline stages must scale independently across multiple processes or when durable message persistence is required for reliability guarantees
- Polling-based status checks where downstream stages periodically query for completion rather than receiving event notifications (rejected)
  Rejected because: Polling introduces unnecessary latency, increases database load with repeated status queries, and complicates error detection compared to immediate event-driven notifications
  When valid: May be necessary when integrating with external systems that do not support webhook or event-based callbacks

## Risks

- Unhandled promise rejections or missing error event handlers can cause silent failures where archive generation fails without proper cleanup or user notification
  Mitigation: Implement comprehensive error event handlers on all stream objects, use promise catch chains consistently, and configure runtime to fail fast on unhandled rejections during development
  Owner: engineering team
- Event handler memory leaks can occur if listeners are not properly removed when streams are destroyed, causing gradual memory growth in long-running processes
  Mitigation: Use once() for single-fire events where appropriate, ensure removeListener calls in cleanup paths, and implement periodic memory profiling in staging environments
  Owner: engineering team
- Race conditions between concurrent event handlers updating shared state can cause inconsistent progress tracking or duplicate processing
  Mitigation: Design event handlers to be idempotent where possible, use atomic cache operations for progress updates, and implement optimistic locking for database state transitions
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
- When implementing stream-based pipelines, register error handlers before piping streams together to ensure errors are caught during the pipe setup phase, and always implement cleanup logic in error handlers to prevent resource leaks
- For cloud service client operations, wrap send calls in try-catch blocks and implement exponential backoff retry logic for transient failures, while distinguishing between retryable and terminal errors based on service-specific error codes
- Structure progress metadata updates to be independent of pipeline success or failure, ensuring that partial progress is recorded even when operations are aborted, to support resume functionality and debugging

## Continuation Context


Verify commands:
- Discover the project's test execution configuration and run the test suite covering archive generation workflows to verify event handler registration and error propagation
- Locate the project's static analysis or linting configuration and execute checks that detect unhandled promise rejections and missing error event handlers
- Identify the project's integration test suite and execute tests that simulate stream errors to verify cleanup operations including upload abortion and resource destruction

Accept when:
- All archive processing pipelines register error, entry, and completion event handlers before initiating stream operations, verified by code inspection or static analysis
- Error injection tests demonstrate that failures in any pipeline stage trigger coordinated cleanup including upload abortion and stream destruction without resource leaks
- Integration tests confirm that progress metadata updates occur independently of pipeline success and that final state correctly reflects completion or failure status

## Enforcement

- Verified by: Code review checklist requiring verification of event handler registration patterns and error propagation logic in all stream-based processing code
- Verified by: Static analysis rules detecting missing error event handlers on stream objects and unhandled promise rejections in asynchronous operations
- Verified by: Integration test suite coverage requirements for error scenarios including stream failures, cloud service errors, and concurrent operation conflicts
- Violation handling: Pull requests introducing stream-based processing without complete error event handlers are blocked until handlers are added and tested
- Violation handling: Static analysis violations for unhandled promises or missing error handlers trigger build failures in continuous integration pipelines
- Violation handling: Production monitoring alerts on unhandled rejection events trigger incident response and require root cause analysis with remediation
- Exception process: Exceptions for simplified error handling in prototype or experimental code must be documented in code comments with a tracking issue for production-ready implementation
- Exception process: Temporary exceptions during migration of legacy synchronous code to event-driven patterns require an approved migration plan with timeline and rollback strategy
- Exception process: All exceptions must be reviewed and approved by a senior engineer familiar with event-driven architecture patterns and their failure modes