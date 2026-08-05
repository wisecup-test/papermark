# Adopt Asynchronous Task Orchestration for Long-Running Archive Operations: Asynchronous Helper Functions

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The system processes dataroom freeze operations that involve archiving multiple documents, generating CSV reports, and uploading large files to object storage, operations that exceed typical HTTP request timeouts
- Archive operations require coordinating multiple asynchronous workflows including batch processing of documents, streaming hash computation, multipart uploads, and progress tracking through metadata updates
- The codebase integrates with serverless compute services and object storage APIs that expose asynchronous interfaces requiring explicit coordination patterns
- Progress reporting requirements necessitate incremental metadata updates during long-running operations to provide user feedback on archive creation status

## Problem Statement

Long-running archive operations that involve document collection, batch processing, streaming compression, hash computation, and multipart uploads cannot complete within synchronous request-response cycles and require explicit concurrency coordination to manage multiple asynchronous workflows, error propagation, resource cleanup, and progress tracking.

## Decision

1. SHOULD: Asynchronous helper functions for CSV generation and intermediate file cleanup SHOULD be extracted as separate composable units

## Policy Block

- SHOULD Asynchronous helper functions for CSV generation and intermediate file cleanup SHOULD be extracted as separate composable units

In scope:
- Archive generation tasks that process multiple documents and produce compressed artifacts
- Operations involving multipart uploads to object storage with streaming hash computation
- Long-running workflows that require progress tracking through metadata updates
- Batch processing operations that coordinate multiple asynchronous data retrieval and transformation steps

Out of scope:
- Synchronous API endpoints that complete within standard HTTP timeout windows
- Simple CRUD operations that do not involve streaming or batch processing
- Client-side asynchronous operations in browser or mobile contexts
- Real-time streaming operations that require WebSocket or server-sent event protocols

## Rationale

- The evidence shows explicit use of asynchronous task orchestration patterns with progress metadata updates at checkpoints (0.05, 0.1, per-batch), indicating a deliberate design to handle operations exceeding synchronous execution limits
- Stream-based hash computation using transform streams and event-driven error handling demonstrates a memory-efficient approach to processing large archives without buffering entire contents
- The pattern of extracting helper functions for CSV generation and cleanup operations alongside batch processing with size-based partitioning reflects a composable concurrency model that separates concerns while coordinating multiple asynchronous workflows
- Integration with serverless compute clients and object storage SDKs with multipart upload support indicates architectural alignment with cloud-native asynchronous execution models

## Consequences

Positive:
- Archive operations can process arbitrarily large document sets without hitting request timeout limits or memory constraints through streaming and batch processing
- Users receive incremental progress feedback during long-running operations through metadata checkpoint updates
- Memory footprint remains bounded through streaming interfaces for compression and upload rather than buffering entire archives
- Error propagation and cleanup logic ensures failed operations abort multipart uploads and release resources properly

Negative:
- Asynchronous task orchestration increases implementation complexity compared to synchronous request handlers, requiring explicit error handling and cleanup coordination
- Debugging failures in asynchronous workflows is more challenging due to distributed execution context and event-driven error propagation
- Progress tracking through metadata updates introduces additional database writes and potential consistency challenges if operations fail mid-execution
- Dependency on external task orchestration infrastructure creates operational complexity and potential points of failure beyond the application code

## Alternatives

- Implement archive operations as synchronous HTTP endpoints with extended timeout configurations (rejected)
  Rejected because: Synchronous endpoints cannot reliably handle operations that may take minutes to complete due to infrastructure timeout limits, and provide no mechanism for incremental progress reporting or graceful handling of client disconnections
  When valid: Only viable for archive operations guaranteed to complete within 30-60 seconds with small document sets
- Use message queue workers with polling-based status checks instead of task orchestration framework (rejected)
  Rejected because: Message queue workers require additional infrastructure for status persistence and polling, increasing operational complexity without providing the integrated progress tracking and error handling that task orchestration frameworks offer
  When valid: Appropriate when the system already has message queue infrastructure and does not require fine-grained progress reporting
- Pre-generate archives on document upload and cache results rather than generating on-demand (deferred)
  Rejected because: Pre-generation eliminates the need for long-running operations but requires storage for cached archives and invalidation logic when documents change
  When valid: Viable optimization if archive requests are frequent and document sets are relatively static

## Risks

- Task orchestration infrastructure failures or quota limits may prevent archive operations from completing, leaving datarooms in incomplete freeze states
  Mitigation: Implement retry logic with exponential backoff, monitor task execution metrics, and provide manual retry mechanisms for failed operations
  Owner: engineering team
- Incomplete error handling in asynchronous workflows may leak resources such as incomplete multipart uploads or orphaned intermediate files
  Mitigation: Ensure all asynchronous operations register error handlers that trigger cleanup, implement periodic cleanup jobs for orphaned resources, and add monitoring for resource leaks
  Owner: engineering team
- Progress metadata updates may fail independently of the archive operation, causing user-facing progress indicators to become stale or inaccurate
  Mitigation: Treat metadata update failures as non-fatal warnings, implement timeout-based progress estimation as fallback, and ensure final operation status is always persisted
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
- Implement archive operations as task functions that accept payload objects containing dataroom identifiers and team context, and coordinate multiple asynchronous workflows including document retrieval, batch processing, streaming compression, hash computation, and multipart upload
- Use transform streams to compute cryptographic hashes incrementally during archive creation and upload, piping archive output through hash computation transforms before passing to upload streams to avoid multiple passes over data
- Structure batch processing to partition documents by cumulative size rather than count alone, ensuring batches remain within memory and processing time constraints while providing meaningful progress increments
- Register error handlers on all event emitters and streams to propagate failures through the workflow, ensuring cleanup operations such as upload abortion execute before task failure

## Continuation Context


Verify commands:
- Discover the project's task orchestration configuration and identify the task definition for dataroom freeze archive operations
- Locate the test suite covering archive task execution and verify tests exist for batch processing, progress updates, error handling, and cleanup on failure
- Identify the project's integration test infrastructure and execute tests that verify end-to-end archive generation with multipart upload and hash computation

Accept when:
- Archive task tests pass including scenarios for single-batch and multi-batch processing with correct progress metadata updates at each checkpoint
- Error handling tests verify that stream errors trigger upload abortion and cleanup operations execute successfully
- Integration tests confirm that generated archives contain all expected documents, computed hashes match archive contents, and final metadata persists correctly

## Enforcement

- Verified by: Code review verification that new long-running operations follow asynchronous task patterns with streaming interfaces
- Verified by: Automated test coverage requirements for asynchronous workflows including error handling and cleanup paths
- Verified by: Architecture review for operations exceeding expected synchronous execution time to ensure task orchestration adoption
- Violation handling: Pull requests implementing long-running operations as synchronous endpoints are rejected with guidance to adopt task orchestration patterns
- Violation handling: Operations that buffer entire archives in memory rather than using streaming interfaces are flagged in code review for refactoring
- Violation handling: Missing error handlers on asynchronous operations are identified through static analysis and required to be added before merge
- Exception process: Operations with proven completion times under 10 seconds may use synchronous implementations with documented justification
- Exception process: Prototype or experimental features may defer full asynchronous implementation with explicit technical debt tracking
- Exception process: Exception requests must include performance testing data demonstrating operation characteristics and be approved by technical lead