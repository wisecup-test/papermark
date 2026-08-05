# Adopt Modular Core Library Architecture with Observability and Event-Driven Boundaries: Public Contracts Defined

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The codebase exhibits a pattern of importing core infrastructure libraries from a centralized internal path, indicating a deliberate separation between feature code and foundational services.
- Observability is implemented through structured logging with contextual metadata, suggesting a need for operational visibility into long-running or complex workflows.
- Event-driven boundaries are established using stream-based processing with error handling and lifecycle hooks, enabling reactive composition of data transformations.
- Cache layer operations use metadata state management to track progress and status, indicating user-facing workflows that require incremental feedback.
- The architecture coordinates AWS SDK clients for storage and compute orchestration, database access patterns for entity retrieval and updates, and middleware-style stream transformations for data integrity verification.

## Problem Statement

Without a consistent internal structuring pattern, feature implementations risk duplicating infrastructure concerns, scattering observability instrumentation, and creating tightly coupled dependencies between business logic and external service clients. This increases maintenance burden, reduces testability, and makes it difficult to enforce cross-cutting concerns such as error handling, progress tracking, and data integrity verification across the codebase.

## Decision

1. SHOULD: Public API contracts SHOULD be defined as typed payload interfaces and exported task definitions to establish clear boundaries between feature modules and orchestration layers.

## Policy Block

- SHOULD Public API contracts SHOULD be defined as typed payload interfaces and exported task definitions to establish clear boundaries between feature modules and orchestration layers.

In scope:
- Feature modules within the enterprise edition that coordinate external services
- Long-running workflows that require operational visibility and progress tracking
- Data transformation pipelines that process user-generated content or sensitive data
- API task definitions that expose asynchronous operations to orchestration layers

Out of scope:
- Standalone utility functions that perform pure computation without external dependencies
- Configuration modules that only export static values or environment variable mappings
- Type definition files that declare interfaces without runtime behavior

## Rationale

- The evidence shows consistent use of internal library imports for storage configuration and AWS SDK clients, indicating an established pattern for centralizing infrastructure concerns and reducing coupling between features and external services.
- Structured logging with contextual metadata appears at multiple workflow stages, demonstrating a deliberate investment in operational observability that enables debugging and performance analysis in production environments.
- Event-driven stream processing with explicit error handling and cleanup logic reflects a mature approach to managing asynchronous data flows and preventing resource leaks in failure scenarios.
- The separation of database read projections from write operations, combined with explicit relationship loading, shows an optimization strategy that balances query performance with data freshness requirements.

## Consequences

Positive:
- Centralized infrastructure libraries reduce code duplication and make it easier to evolve cross-cutting concerns such as authentication, rate limiting, and retry logic without modifying feature code.
- Structured observability instrumentation provides operational visibility into complex workflows, enabling faster incident response and data-driven performance optimization.
- Event-driven boundaries with explicit error handling improve system resilience by isolating failures and preventing cascading errors across component boundaries.
- Explicit database access patterns with controlled relationship loading reduce over-fetching and improve query performance in high-throughput scenarios.

Negative:
- The internal library abstraction introduces an additional layer of indirection that developers must understand, potentially increasing onboarding time for new team members.
- Structured logging with rich contextual metadata increases log volume and storage costs, requiring investment in log aggregation and retention policies.
- Event-driven stream processing with middleware transformations adds complexity to data flow debugging, as errors may occur in multiple pipeline stages.
- The requirement to discover and verify exact library versions before use adds friction to development workflows, though it improves long-term maintainability and reduces version-related bugs.

## Alternatives

- Allow feature modules to directly instantiate external service clients without going through internal library abstractions. (rejected)
  Rejected because: Direct instantiation scatters configuration logic across the codebase, making it difficult to enforce consistent retry policies, timeout settings, and credential management. The evidence shows a deliberate pattern of centralizing these concerns.
  When valid: In prototype or proof-of-concept code where long-term maintainability is not a priority.
- Use synchronous blocking operations instead of event-driven stream processing for data transformations. (rejected)
  Rejected because: Synchronous operations would block the event loop during large file processing, degrading system responsiveness. The evidence shows stream-based processing with hash computation, indicating a need to handle large data volumes efficiently.
  When valid: For small data payloads where the overhead of stream setup exceeds the processing time.
- Implement observability through external APM agents rather than explicit structured logging. (deferred)
  Rejected because: Not rejected; this is complementary. External APM provides automatic instrumentation but may not capture domain-specific context. The evidence shows explicit logging with business-relevant metadata that APM agents cannot infer.
  When valid: As a complementary approach to capture low-level performance metrics and distributed traces alongside domain-specific structured logs.

## Risks

- Internal library abstractions may become bottlenecks if they do not expose sufficient configuration options for advanced use cases, forcing developers to work around the abstraction layer.
  Mitigation: Design internal libraries with extension points and escape hatches that allow advanced users to access underlying clients when necessary, while maintaining the default path through the abstraction.
  Owner: Platform Engineering Team
- Structured logging with rich contextual metadata may inadvertently log sensitive user data, creating compliance and privacy risks.
  Mitigation: Implement log sanitization middleware that redacts sensitive fields before emission, and establish code review guidelines that require explicit justification for logging user-generated content.
  Owner: Security and Compliance Team
- Event-driven stream processing with multiple error handlers may mask underlying issues if errors are caught and logged but not propagated, leading to silent failures.
  Mitigation: Establish a convention that error handlers must either propagate errors to upstream handlers or emit metrics that trigger alerting, ensuring that failures are visible to operations teams.
  Owner: Engineering Team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When implementing new feature modules, begin by identifying the internal library path for infrastructure concerns by examining existing feature imports. Establish a consistent import pattern across the module to signal architectural boundaries.
- For workflows that require progress tracking, design the cache metadata schema upfront to include both numeric progress indicators and human-readable status text. Ensure that progress updates are idempotent to handle retry scenarios.
- When implementing stream-based transformations, use the pipeline pattern to compose multiple transformation stages, and ensure that each stage registers its own error handler to provide granular failure diagnostics.

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the linting rules that enforce import path conventions for core library usage.
- Locate the project's test suite directory and run integration tests that verify structured logging output contains required contextual metadata fields.
- Identify the project's dependency verification script and execute it to confirm that all versioned libraries match the resolved versions in the lock artifact.

Accept when:
- Static analysis passes with no violations of core library import path conventions, confirming that feature modules do not directly instantiate external service clients.
- Integration tests confirm that all long-running workflows emit structured log events at key lifecycle points with the required contextual metadata fields.
- Dependency verification confirms that the runtime environment matches the exact versions specified in the project's lock artifact, and no undeclared dependencies are present.

## Enforcement

- Verified by: Automated static analysis in continuous integration that enforces import path conventions and detects direct instantiation of external service clients.
- Verified by: Code review checklist that requires reviewers to verify structured logging at workflow lifecycle points and confirm error handling in event-driven boundaries.
- Verified by: Dependency audit tooling that validates runtime library versions against the lock artifact and fails the build on version mismatches.
- Violation handling: Build failures block merge to main branch when static analysis detects import path violations or missing error handlers in stream processing code.
- Violation handling: Code review feedback requires authors to add structured logging or refactor direct service client instantiation before approval.
- Violation handling: Dependency version mismatches trigger automated alerts to the platform engineering team and block deployment until resolved.
- Exception process: Developers may request an exception to import path conventions by documenting the technical justification in an architecture decision log entry and obtaining approval from the platform engineering lead.
- Exception process: Exceptions to structured logging requirements may be granted for performance-critical hot paths where logging overhead is measured and documented as unacceptable.
- Exception process: Temporary exceptions to dependency version requirements may be granted during security incident response when a patched version is not yet available in the lock artifact, with a mandatory follow-up task to update the lock file.