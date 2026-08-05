# Adopt Structured Logging with Contextual Metadata for Asynchronous Archive Operations: Log Messages Include

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The system performs long-running asynchronous archive operations that process multiple documents, generate batches, and interact with cloud storage services, requiring visibility into operation progress and state transitions.
- Archive operations involve multiple stages including document collection, batch creation, compression, upload, and metadata persistence, each requiring distinct observability markers for debugging and monitoring.
- The archive workflow coordinates between database queries, stream processing, cloud storage uploads, and lambda invocations, creating a distributed execution context that must be traceable across boundaries.
- Progress tracking and error diagnosis in multi-batch operations require structured metadata that captures batch numbers, file counts, size metrics, and completion states at each stage.

## Problem Statement

Asynchronous archive operations spanning multiple execution stages, batch processing, and external service interactions require consistent observability that captures operational context, progress metrics, and state transitions without relying on unstructured log messages that are difficult to query, aggregate, or correlate across distributed execution boundaries.

## Decision

1. MAY: Log messages MAY include conditional text formatting based on operation characteristics such as single versus multi-batch processing.

## Policy Block

- MAY Log messages MAY include conditional text formatting based on operation characteristics such as single versus multi-batch processing.

In scope:
- Asynchronous archive generation tasks
- Multi-stage document processing workflows
- Batch processing operations with progress tracking
- Operations involving cloud storage uploads and lambda invocations
- CSV generation and export operations

Out of scope:
- Synchronous request-response logging
- Authentication and authorization audit logs
- Database query performance logging
- Client-side application logging

## Rationale

- The evidence shows consistent use of structured metadata objects passed to logger methods across all stages of the archive operation, enabling queryable and aggregatable observability data.
- The pattern separates progress tracking concerns through a dedicated metadata mechanism that updates numeric progress and status text independently from log output, supporting both machine and human consumption.
- Batch processing operations demonstrate systematic logging of batch context including sequence numbers, file counts, and size metrics, enabling operational analysis of batch distribution and performance characteristics.
- The structured approach facilitates correlation of log events across distributed execution boundaries by including consistent identifiers and contextual metadata in each log statement.

## Consequences

Positive:
- Structured metadata enables efficient querying, filtering, and aggregation of log data across distributed operations without parsing unstructured text.
- Progress tracking through dedicated metadata mechanism provides real-time visibility into long-running operations for both monitoring systems and end users.
- Batch-level metadata facilitates performance analysis, capacity planning, and debugging of multi-batch processing workflows.
- Consistent inclusion of operational context in log statements enables correlation of events across service boundaries and execution stages.

Negative:
- Structured logging requires additional development effort to construct metadata objects for each log statement compared to simple string interpolation.
- Metadata construction adds minor runtime overhead for object allocation and serialization in high-frequency logging scenarios.
- Inconsistent metadata schemas across different operation types can reduce the effectiveness of log aggregation and analysis tools.
- Progress metadata mechanism introduces coupling between operation logic and progress tracking infrastructure.

## Alternatives

- Use unstructured string interpolation for all log messages with embedded context values (rejected)
  Rejected because: Unstructured logs require regex parsing or full-text search for analysis, making it difficult to efficiently query specific metrics, aggregate batch statistics, or correlate events across distributed execution boundaries.
  When valid: Simple single-stage operations with minimal context where log analysis requirements are limited to manual inspection.
- Emit structured events to a dedicated event stream separate from application logs (deferred)
  Rejected because: Not rejected; represents a complementary approach that could be adopted alongside structured logging for higher-volume operational telemetry.
  When valid: When operational event volume exceeds log system capacity or when event-driven monitoring and alerting infrastructure is available.
- Store progress and operational state exclusively in database records without logging (rejected)
  Rejected because: Database-only state tracking lacks the temporal resolution and diagnostic context needed for debugging failures, performance analysis, and real-time monitoring of in-flight operations.
  When valid: When only final operation outcomes matter and intermediate state transitions do not require observability.

## Risks

- Inconsistent metadata schemas across different operations reduce the effectiveness of log aggregation and querying tools.
  Mitigation: Define standard metadata schema conventions for common operation types and enforce through code review and linting rules.
  Owner: engineering team
- High-frequency logging with complex metadata objects may introduce performance overhead in critical paths.
  Mitigation: Profile logging overhead in performance-sensitive code paths and apply sampling or async logging for high-frequency events.
  Owner: engineering team
- Progress metadata mechanism may become out of sync with actual operation state if updates are missed or fail silently.
  Mitigation: Implement progress update error handling and validation to ensure metadata updates succeed or trigger alerts on failure.
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
- Define standard metadata schema conventions for common operation types including required fields for identifiers, counts, sizes, and timestamps to ensure consistency across the codebase.
- Implement progress metadata updates at key operation milestones with error handling to ensure updates succeed or trigger alerts, preventing silent failures that leave progress tracking out of sync.
- For stream-based operations, ensure error handlers log structured error context and trigger cleanup operations such as aborting partial uploads to prevent resource leaks.

## Continuation Context


Verify commands:
- Discover and execute the project's static analysis configuration to verify that logging statements in multi-stage operations include structured metadata objects.
- Discover and run the project's test suite to verify that progress metadata updates occur at expected operation milestones and include both numeric progress and status text.
- Discover and execute the project's log validation tooling to verify that batch processing operations emit batch-level metadata including sequence numbers, counts, and size metrics.

Accept when:
- All logging statements in multi-stage asynchronous operations include structured metadata objects with relevant operational context.
- Progress tracking mechanism updates both numeric progress indicators and status text at key operation milestones.
- Batch processing operations emit structured metadata including batch number, total batches, file counts, and size metrics at batch start and completion.

## Enforcement

- Verified by: Code review verification that new logging statements include structured metadata objects with appropriate operational context.
- Verified by: Static analysis rules that flag logging statements missing structured metadata in designated operation types.
- Verified by: Integration test validation that progress metadata updates occur at expected milestones with correct values.
- Violation handling: Code review feedback requiring addition of structured metadata to logging statements before merge approval.
- Violation handling: Static analysis failures blocking continuous integration pipeline until logging statements are corrected.
- Violation handling: Post-deployment monitoring alerts for operations with missing or malformed log metadata triggering remediation tasks.
- Exception process: Document justification for unstructured logging in specific contexts where structured metadata is impractical or unnecessary.
- Exception process: Obtain approval from technical lead for exceptions in performance-critical paths where logging overhead is measured and documented.
- Exception process: Record approved exceptions in architecture decision log with expiration date for re-evaluation.