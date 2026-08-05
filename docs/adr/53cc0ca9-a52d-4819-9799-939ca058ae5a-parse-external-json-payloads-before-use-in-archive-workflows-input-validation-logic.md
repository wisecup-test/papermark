# Parse External JSON Payloads Before Use in Archive Workflows: Input Validation Logic

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The dataroom freeze archive workflow receives payloads from external sources including decoded Lambda invocation results and trigger payloads that must be deserialized before processing
- Archive generation coordinates multiple asynchronous operations including document collection, batch creation, streaming uploads, and metadata updates that require validated input structures
- The workflow integrates with cloud storage services and Lambda functions where payload integrity directly affects data consistency and operational correctness
- Input validation through explicit parsing establishes a trust boundary between external data sources and internal processing logic, preventing malformed data from propagating through the archive pipeline

## Problem Statement

External payloads arriving from Lambda invocations, trigger events, and decoded sources must be validated and parsed before use in archive workflows to prevent runtime errors, data corruption, and security vulnerabilities from malformed or malicious input structures.

## Decision

1. SHOULD: Input validation logic SHOULD be positioned at the earliest possible point in the request processing pipeline to establish clear trust boundaries

## Policy Block

- SHOULD Input validation logic SHOULD be positioned at the earliest possible point in the request processing pipeline to establish clear trust boundaries

In scope:
- Lambda invocation result payloads containing response bodies
- Trigger event payloads received from external event sources
- Decoded payload strings from base64 or other encoding schemes
- Any external data structure that will be used in database queries, storage operations, or workflow coordination

Out of scope:
- Internal data structures already validated and constructed within the application boundary
- Configuration objects loaded from trusted internal sources at application startup
- Type-safe objects returned from internal function calls within the same module

## Rationale

- The evidence shows explicit JSON.parse calls on decodedPayload and lambdaResult.body, indicating a deliberate validation boundary between external sources and internal processing
- The workflow coordinates multiple cloud services including storage uploads, Lambda invocations, and database updates where input integrity is critical for data consistency
- Parsing external payloads at ingress points prevents type confusion, injection attacks, and runtime errors that could corrupt archive generation or metadata persistence
- The pattern aligns with defense-in-depth principles by establishing explicit trust boundaries at system integration points

## Consequences

Positive:
- Prevents malformed JSON from causing runtime errors or data corruption in archive generation workflows
- Establishes clear trust boundaries between external event sources and internal processing logic
- Enables early detection of payload structure mismatches before data reaches database or storage layers
- Provides explicit validation points where schema enforcement and error logging can be centralized

Negative:
- Adds parsing overhead to every external payload ingress point, increasing latency for high-frequency workflows
- Requires maintenance of validation logic as payload schemas evolve across Lambda functions and trigger sources
- May introduce failure modes if parsing logic is not properly synchronized with upstream payload producers

## Alternatives

- Trust external payloads without explicit parsing and rely on runtime type coercion (rejected)
  Rejected because: Runtime type coercion masks validation errors and allows malformed data to propagate through the system, increasing risk of data corruption and security vulnerabilities
  When valid: Never valid for external payloads from untrusted or loosely-coupled sources
- Implement schema validation using a dedicated validation library with type guards (deferred)
  Rejected because: Not rejected; represents a more robust evolution of the current pattern
  When valid: When payload complexity increases or when stronger type safety guarantees are required across multiple integration points
- Validate payloads only at database or storage operation boundaries (rejected)
  Rejected because: Defers validation until after business logic has executed, allowing invalid data to affect workflow state and making error recovery more complex
  When valid: Only when external payloads are already validated by a trusted API gateway or middleware layer

## Risks

- JSON.parse throws synchronous exceptions on malformed input, potentially crashing the workflow if not properly caught
  Mitigation: Wrap all JSON.parse calls in try-catch blocks with error logging and graceful degradation paths
  Owner: engineering team
- Payload schema drift between Lambda producers and consumers can cause validation failures in production
  Mitigation: Implement integration tests that validate payload contracts across service boundaries and establish schema versioning conventions
  Owner: engineering team
- Parsing overhead may become a performance bottleneck for high-volume archive workflows
  Mitigation: Profile parsing performance in production and consider streaming parsers or payload size limits if latency becomes problematic
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
- Position JSON parsing immediately after receiving external payloads and before any property access or business logic execution to establish the earliest possible validation boundary
- Implement structured error logging for parse failures that includes payload source, expected schema version, and sufficient context for debugging without exposing sensitive payload content
- Consider extracting validation logic into reusable functions or middleware that can be consistently applied across all external integration points in the archive workflow

## Continuation Context


Verify commands:
- Discover the project's test runner from the dependency manifest and execute the test suite covering archive workflow payload handling
- Locate the project's static analysis configuration and run type checking to verify payload parsing occurs before property access
- Identify the project's linting configuration and verify no direct property access on unparsed external payloads is flagged

Accept when:
- All external JSON payloads from Lambda results and trigger events are explicitly parsed before use
- Parse operations include error handling that prevents malformed data from reaching database or storage operations
- Static analysis confirms no direct property access on unparsed external payload variables

## Enforcement

- Verified by: Code review checklist requiring explicit parsing of all external payloads at integration boundaries
- Verified by: Static analysis rules detecting unparsed external data usage in database or storage operations
- Verified by: Integration tests validating error handling for malformed payloads from Lambda and trigger sources
- Violation handling: Code review blocks merge if external payloads are used without explicit parsing
- Violation handling: Static analysis failures in continuous integration prevent deployment
- Violation handling: Runtime parse errors trigger alerts and workflow termination to prevent data corruption
- Exception process: Document the trusted source and validation guarantees that make parsing unnecessary
- Exception process: Obtain architecture review approval demonstrating equivalent validation at an upstream boundary
- Exception process: Add inline comments explaining the exception rationale and upstream validation mechanism