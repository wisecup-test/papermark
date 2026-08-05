# Export Typed Task Payloads as Public API Contracts: Task Implementations That

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- Background task orchestration requires strongly-typed payload contracts to coordinate asynchronous work across service boundaries and runtime environments
- The codebase exports FreezeArchivePayload and dataroomFreezeArchiveTask as public contracts, establishing a typed interface for dataroom freeze archive operations invoked via SDK-based task triggers
- Archive generation workflows coordinate multiple subsystems including storage clients, streaming upload mechanisms, database queries, and serverless function invocation, necessitating explicit contract boundaries
- Observability instrumentation records structured metadata at multiple workflow stages, requiring consistent payload shapes for correlation and debugging across distributed execution contexts

## Problem Statement

Asynchronous background tasks that span multiple runtime environments and coordinate complex workflows require explicit, versioned API contracts to ensure type safety, enable independent evolution of task producers and consumers, and provide clear boundaries for testing and observability instrumentation.

## Decision

1. MUST: Task implementations that depend on versioned SDK libraries SHALL require consumers to discover the exact resolved version from the project's lock artifact before integration

## Policy Block

- MUST Task implementations that depend on versioned SDK libraries SHALL require consumers to discover the exact resolved version from the project's lock artifact before integration

In scope:
- Background task definitions that coordinate multi-step workflows
- Task payloads invoked across process or runtime boundaries
- Asynchronous operations requiring progress tracking or observability correlation
- Task contracts consumed by external orchestration systems or SDKs

Out of scope:
- Synchronous function calls within a single process boundary
- Internal helper functions not exposed as public API
- Ephemeral data structures used only within a single execution context

## Rationale

- The evidence shows FreezeArchivePayload and dataroomFreezeArchiveTask exported as public contracts, establishing a typed boundary for task invocation that enables independent testing and evolution of task producers and consumers
- The workflow coordinates storage clients, streaming uploads, database queries, and serverless function invocation, requiring explicit contracts to manage complexity and enable observability across distributed execution
- Structured logging at workflow stages uses consistent metadata shapes derived from the task payload, enabling correlation and debugging across asynchronous execution boundaries
- Exporting task contracts as public API enables type-safe integration with SDK-based orchestration systems while maintaining clear boundaries for versioning and compatibility

## Consequences

Positive:
- Type-safe task invocation prevents runtime errors from payload shape mismatches
- Explicit contracts enable independent evolution and testing of task producers and consumers
- Observability instrumentation can rely on stable payload shapes for correlation and structured logging
- Clear API boundaries facilitate documentation, versioning, and backward compatibility management

Negative:
- Public contracts create versioning obligations and require careful management of breaking changes
- Additional ceremony required to define and export payload types for all background tasks
- Contract evolution may require coordination across multiple consumers and runtime environments

## Alternatives

- Use untyped or loosely-typed payloads with runtime validation (rejected)
  Rejected because: Runtime validation defers error detection and provides no compile-time safety, increasing the risk of production failures and reducing developer confidence in refactoring
  When valid: Acceptable for rapid prototyping or one-off scripts where type safety overhead exceeds risk
- Embed task logic inline without explicit contract boundaries (rejected)
  Rejected because: Inline task logic couples invocation to implementation, preventing independent testing, observability instrumentation, and evolution of orchestration mechanisms
  When valid: Suitable for simple synchronous operations within a single process boundary
- Generate task contracts from schema definitions or OpenAPI specifications (deferred)
  Rejected because: Not rejected; schema-driven generation could complement explicit exports but requires additional tooling and workflow integration
  When valid: When task contracts must align with external API specifications or cross-language boundaries

## Risks

- Breaking changes to public task contracts may impact multiple consumers across runtime environments without immediate detection
  Mitigation: Implement contract testing and versioning strategies; use semantic versioning for task contract packages; maintain backward compatibility or provide migration paths
  Owner: engineering team
- Payload type definitions may drift from actual runtime usage if validation is not enforced at task boundaries
  Mitigation: Implement runtime validation that enforces payload type constraints; use schema validation libraries to verify payloads match exported contracts at task entry points
  Owner: engineering team
- SDK version mismatches between task definition and invocation environments may cause runtime failures despite type safety
  Mitigation: Enforce lock-version grounding policy; document SDK version requirements in task contracts; implement version compatibility checks at task registration
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
- Define task payload interfaces in dedicated contract modules separate from implementation logic to enable independent testing and versioning; export both the payload type and task identifier as public API
- Implement runtime validation at task entry points to verify payloads conform to exported contracts; use schema validation to catch type drift and provide clear error messages for contract violations
- Structure task implementations to emit observability events at workflow stage boundaries using consistent metadata shapes derived from the task payload; ensure correlation identifiers flow through all subsystem interactions

## Continuation Context


Verify commands:
- Discover the project's type-checking configuration and execute the type checker to verify all task payload exports are well-typed and all task invocations pass type validation
- Locate the project's test suite and execute contract tests that verify task payloads conform to exported type definitions and runtime validation enforces contract constraints
- Identify the project's linting or static analysis tooling and verify it detects untyped task payloads or missing contract exports

Accept when:
- Type checker reports no errors for task payload definitions and all task invocation sites pass type validation
- Contract tests verify runtime validation rejects payloads that violate exported type constraints
- Static analysis confirms all background task definitions export typed payload interfaces and task identifiers as public API

## Enforcement

- Verified by: Type checking in continuous integration pipeline
- Verified by: Contract tests validating payload shape conformance
- Verified by: Code review verification of public API exports for new task definitions
- Violation handling: Type check failures block merge to protected branches
- Violation handling: Contract test failures trigger build failure and prevent deployment
- Violation handling: Code review process requires explicit approval for task contract changes
- Exception process: Document rationale for untyped payloads or missing contracts in architectural decision log
- Exception process: Obtain approval from technical lead or architecture review board
- Exception process: Create tracking issue for future contract migration with acceptance criteria