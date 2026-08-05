# Use Metadata Cache Layer for Long-Running Archive Task Progress Tracking: Progress Metadata Include

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Long-running archive generation tasks require real-time progress visibility for client applications monitoring task execution state
- The dataroom freeze archive workflow processes multiple batches of documents with varying sizes, requiring incremental progress updates at distinct workflow stages
- Structured logging captures operational events but does not provide queryable state for external consumers tracking task completion
- A cache layer provides low-latency read access to ephemeral progress metadata without persisting transient state to the primary database

## Problem Statement

Archive generation workflows span multiple asynchronous operations including document collection, batch creation, S3 uploads, and CSV generation. Client applications need to display real-time progress percentages and status text to end users, but structured logging alone does not expose queryable state. A mechanism is required to store and retrieve ephemeral progress metadata with sub-second latency while avoiding unnecessary writes to the primary persistence layer.

## Decision

1. MUST: Progress metadata MUST include numeric progress values and human-readable status text describing the current operation

## Policy Block

- MUST Progress metadata MUST include numeric progress values and human-readable status text describing the current operation

In scope:
- Archive generation tasks triggered through the task orchestration framework
- Multi-stage workflows requiring client-visible progress tracking
- Ephemeral metadata with read-heavy access patterns and short lifecycle

Out of scope:
- Final archive URLs and hashes persisted to the primary database
- Audit logs and operational events recorded through structured logging
- Synchronous request-response operations completing within HTTP timeout windows

## Rationale

- The evidence shows metadata.set operations updating progress and text fields at five distinct workflow stages with values ranging from 0.05 to 0.1, demonstrating incremental progress tracking
- Parallel structured logging with logger.info calls captures operational context including batch details, file counts, and size metrics for debugging and audit purposes
- The dataroom.update operation persists only final archive results to the database, separating ephemeral progress state from durable business data
- The pattern supports responsive user interfaces by providing queryable progress state without polling structured logs or database records

## Consequences

Positive:
- Client applications can poll cache layer endpoints to display real-time progress bars and status messages with sub-second latency
- Primary database write load remains bounded to final state transitions rather than frequent progress updates
- Ephemeral progress metadata automatically expires without requiring explicit cleanup logic
- Structured logging remains available for post-hoc analysis and debugging without serving real-time queries

Negative:
- Introduces dependency on cache layer infrastructure requiring separate deployment, monitoring, and failure handling
- Progress metadata may become inconsistent with actual task state if cache writes fail silently
- Developers must maintain two parallel update paths for logging and cache layer metadata
- Cache layer outages degrade user experience even when core archive generation succeeds

## Alternatives

- Poll structured logs through log aggregation query APIs to extract progress information (rejected)
  Rejected because: Log query APIs introduce 1-5 second latency and are not designed for high-frequency polling by client applications
  When valid: Acceptable for batch analytics or post-hoc progress reconstruction where real-time visibility is not required
- Write progress updates directly to database task status table with indexed queries (rejected)
  Rejected because: Generates excessive write load on primary database for ephemeral state that does not require durability guarantees
  When valid: Appropriate when progress state must survive process restarts or requires transactional consistency with business data
- Stream progress events through WebSocket or Server-Sent Events without persisting intermediate state (deferred)
  Rejected because: Requires persistent client connections and does not support clients joining mid-workflow to query current progress
  When valid: Viable for real-time dashboards with guaranteed client connectivity throughout task execution

## Risks

- Cache layer failures cause progress tracking to fail silently while archive generation continues, creating user confusion
  Mitigation: Implement cache write error logging and fallback to database-backed progress table when cache is unavailable
  Owner: engineering team
- Progress values may not accurately reflect actual completion percentage if batch sizes vary significantly or unexpected errors occur
  Mitigation: Calculate progress weights based on actual batch sizes and file counts rather than uniform stage increments
  Owner: engineering team
- Cache layer memory pressure from concurrent long-running tasks may cause eviction of active progress metadata
  Mitigation: Configure appropriate TTL values and memory limits based on expected task concurrency and duration profiles
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
- Define progress stage boundaries based on workflow analysis to ensure monotonic progress values that sum to 1.0 across all stages
- Use task identifiers as cache keys with namespacing to prevent collisions between concurrent archive generation tasks
- Implement cache write timeouts and circuit breakers to prevent cache layer latency from blocking archive generation workflow execution

## Continuation Context


Verify commands:
- Discover the project's test suite configuration and execute integration tests covering archive task progress tracking workflows
- Locate cache layer health check endpoints in the deployment configuration and verify connectivity from task execution environment
- Identify monitoring dashboards tracking cache hit rates and write latencies for progress metadata operations

Accept when:
- Integration tests demonstrate progress metadata updates at all defined workflow stages with correct values and status text
- Cache layer health checks return successful responses with sub-100ms latency from task execution environment
- Monitoring dashboards show cache write success rates above 99.9% for progress metadata operations under normal load

## Enforcement

- Verified by: Automated integration tests validating progress metadata updates at each workflow stage
- Verified by: Code review checklist requiring cache layer updates for new long-running task implementations
- Verified by: Monitoring alerts on cache write error rates exceeding defined thresholds
- Violation handling: CI pipeline fails if integration tests do not verify progress tracking for new archive workflows
- Violation handling: Code review blocks merge if long-running tasks lack cache layer progress updates
- Violation handling: On-call rotation investigates cache write error alerts within defined SLA windows
- Exception process: Short-duration tasks completing within HTTP timeout windows may omit progress tracking with architectural review approval
- Exception process: Workflows with no client-facing progress requirements may use structured logging only with product owner sign-off
- Exception process: Cache layer outages trigger automatic fallback to database-backed progress table without requiring exception approval