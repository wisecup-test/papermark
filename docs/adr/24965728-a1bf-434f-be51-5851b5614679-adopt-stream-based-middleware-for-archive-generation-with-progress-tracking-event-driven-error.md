# Adopt Stream-Based Middleware for Archive Generation with Progress Tracking: Event Driven Error

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The system generates large archive files from dataroom documents, requiring streaming processing to avoid memory exhaustion and enable real-time progress tracking during long-running operations.
- Archive generation involves multiple concurrent concerns: hashing for integrity verification, progress metadata updates for user feedback, multipart upload coordination, and error handling across asynchronous boundaries.
- The implementation uses event-driven stream middleware to intercept and process data chunks as they flow through the archive pipeline, enabling cross-cutting concerns without coupling business logic to infrastructure.
- Progress tracking requires updating external metadata stores at specific milestones while the archive stream is being constructed, necessitating side-effect coordination within the streaming pipeline.
- The pattern coordinates multiple AWS SDK clients for storage and compute orchestration, structured logging for observability, and database updates for persistence, all within a serverless task execution context.

## Problem Statement

How can the system generate large archives with integrity verification, progress tracking, and error recovery while maintaining separation between streaming data transformation, side-effect coordination, and infrastructure concerns across asynchronous boundaries?

## Decision

1. MUST: Event-driven error handlers MUST be attached to all stream sources and archive instances to propagate failures and trigger cleanup operations including upload abortion.

## Policy Block

- MUST Event-driven error handlers MUST be attached to all stream sources and archive instances to propagate failures and trigger cleanup operations including upload abortion.

In scope:
- Archive generation tasks that process multiple documents into compressed formats
- Long-running operations requiring real-time progress feedback to external consumers
- Streaming pipelines that require integrity verification through cryptographic hashing
- Multipart upload workflows coordinating with cloud storage services
- Serverless task execution contexts with event-driven orchestration

Out of scope:
- Synchronous file operations that complete within request-response cycles
- In-memory archive generation for small datasets that do not require streaming
- Static file serving without integrity verification requirements
- Client-side archive generation in browser contexts

## Rationale

- The evidence shows stream middleware patterns with chunk-level callbacks for hashing and progress tracking, enabling separation of concerns between data transformation and side effects.
- Event-driven error handling with archive.on('entry'), archive.on('error'), and source.on('error') demonstrates coordination across asynchronous boundaries required for reliable cleanup and failure propagation.
- Progress metadata updates at specific milestones (0.05, 0.1, batch-specific values) combined with structured logging provide observability and user feedback without coupling to the core streaming logic.
- The pattern coordinates multiple infrastructure concerns (cloud storage clients, compute orchestration, database persistence) through well-defined boundaries while maintaining testability and separation of business logic.

## Consequences

Positive:
- Memory-efficient processing of large archives through streaming eliminates memory exhaustion risks for multi-gigabyte datasets.
- Stream middleware enables cross-cutting concerns like hashing and progress tracking without coupling business logic to infrastructure implementation details.
- Event-driven error handling provides reliable cleanup and failure propagation across asynchronous boundaries, preventing resource leaks.
- Structured logging and progress metadata enable comprehensive observability and real-time user feedback for long-running operations.

Negative:
- Stream middleware introduces complexity in error handling and backpressure management across multiple asynchronous boundaries.
- Coordinating side effects (progress updates, logging, hashing) within streaming pipelines increases cognitive load and testing surface area.
- Event-driven architectures with multiple error handlers require careful ordering and cleanup logic to prevent race conditions and resource leaks.
- Dependency on cloud SDK clients and task orchestration frameworks creates coupling to specific infrastructure providers and versioning constraints.

## Alternatives

- Generate archives in-memory and write complete files to storage in a single operation (rejected)
  Rejected because: In-memory generation causes memory exhaustion for large multi-document archives and prevents real-time progress tracking during generation.
  When valid: Acceptable for small archives under 100MB where memory constraints are not a concern and progress feedback is not required.
- Use polling-based progress tracking by periodically querying archive generation state from external storage (rejected)
  Rejected because: Polling introduces latency in progress updates and requires additional storage operations, increasing complexity and cost without providing real-time feedback.
  When valid: May be appropriate for batch processing systems where near-real-time progress is sufficient and streaming middleware is not available.
- Separate hashing and archive generation into sequential pipeline stages with intermediate storage (rejected)
  Rejected because: Sequential stages require writing and re-reading large files, doubling storage I/O and preventing concurrent hashing during archive generation.
  When valid: Useful when integrity verification must be performed by separate systems or when streaming middleware is not supported by the runtime environment.

## Risks

- Stream backpressure mismanagement can cause memory accumulation if downstream consumers (upload, hashing) cannot keep pace with archive generation.
  Mitigation: Implement proper backpressure handling in transform streams and monitor memory usage metrics during archive generation. Configure appropriate buffer limits and chunk sizes.
  Owner: engineering team
- Error handler ordering and cleanup logic may fail to abort multipart uploads or release resources if exceptions occur during stream processing.
  Mitigation: Establish explicit error handler registration order, implement comprehensive cleanup in error paths, and add integration tests that verify resource cleanup under failure conditions.
  Owner: engineering team
- Progress metadata updates may fail silently if the cache layer becomes unavailable, leaving users without feedback during long-running operations.
  Mitigation: Implement retry logic for metadata updates with exponential backoff, log update failures for monitoring, and ensure archive generation continues even if progress tracking fails.
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
- Stream middleware should be implemented as transform streams with chunk callbacks that update hasher state and invoke progress callbacks at appropriate intervals without blocking the data flow.
- Event handlers for error conditions must be registered before starting stream processing and should coordinate cleanup across all active resources including aborting multipart uploads and destroying stream instances.
- Progress tracking should use fractional values representing pipeline stages and batch progress, with metadata updates occurring at milestone boundaries rather than per-chunk to minimize cache layer load.
- Structured logging should include correlation identifiers, batch numbers, file counts, and size metrics to enable tracing and debugging of long-running archive generation operations across distributed system boundaries.

## Continuation Context


Verify commands:
- Discover the project's test execution configuration and run integration tests that verify stream-based archive generation with progress tracking and error handling.
- Locate the project's linting and static analysis configuration, then execute checks to verify stream middleware implements proper backpressure handling and error propagation.
- Identify the project's observability tooling and verify that structured logging captures archive generation milestones with required contextual metadata.

Accept when:
- Integration tests demonstrate successful archive generation with concurrent hashing, progress updates, and multipart upload coordination for datasets exceeding memory limits.
- Error injection tests verify that stream error handlers properly abort uploads, destroy resources, and propagate failures without resource leaks.
- Progress tracking tests confirm metadata updates occur at expected milestones and structured logs contain batch numbers, file counts, and size metrics for all archive operations.

## Enforcement

- Verified by: Integration tests that exercise archive generation with large datasets and verify memory usage remains bounded
- Verified by: Code review verification that stream middleware implements proper transform stream interfaces with chunk callbacks
- Verified by: Static analysis checks that all stream sources and archive instances have registered error handlers
- Verified by: Observability monitoring that confirms structured logging captures required milestone metadata
- Violation handling: Archive generation implementations that do not use stream-based processing must be refactored before deployment to production environments
- Violation handling: Missing error handlers on stream sources trigger build failures and must be resolved before merge
- Violation handling: Progress tracking implementations that do not update metadata at required milestones must be corrected to meet user feedback requirements
- Violation handling: Structured logging that omits required contextual metadata must be enhanced to meet observability standards
- Exception process: Small archive operations under defined size thresholds may use in-memory generation with approval from the architecture review board
- Exception process: Temporary progress tracking degradation may be accepted during cache layer outages with explicit monitoring and alerting
- Exception process: Alternative hashing strategies may be approved for specific compliance requirements that conflict with stream-based middleware patterns