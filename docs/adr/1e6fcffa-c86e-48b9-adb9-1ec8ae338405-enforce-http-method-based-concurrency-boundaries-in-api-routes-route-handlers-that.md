# Enforce HTTP Method-Based Concurrency Boundaries in API Routes: Route Handlers That

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

## Context

- API route handlers process HTTP requests with distinct methods (POST, GET, PATCH, DELETE) that define operation semantics and concurrency characteristics
- Database access patterns within route handlers include complex queries with joins, selections, and transactional operations that require consistent isolation boundaries
- Input validation using schema parsers occurs before database operations, establishing a validation-then-execution pattern that must maintain atomicity guarantees
- Multiple route handlers share common database entities and access patterns, requiring consistent concurrency control to prevent race conditions and data integrity violations
- Authentication and authorization checks precede business logic, creating a multi-stage request pipeline where concurrency boundaries must be preserved across stages

## Problem Statement

API route handlers that perform database operations without explicit concurrency boundaries risk data races, inconsistent reads, and integrity violations when multiple requests access shared resources. The HTTP method alone does not guarantee safe concurrent execution, particularly for POST, PATCH, and DELETE operations that modify state. Without standardized concurrency control aligned to HTTP semantics, handlers may produce non-deterministic results under load.

## Decision

1. MUST: Route handlers that execute multiple database operations within a single request MUST use database transaction boundaries to ensure atomicity and isolation.

## Policy Block

- MUST Route handlers that execute multiple database operations within a single request MUST use database transaction boundaries to ensure atomicity and isolation.

In scope:
- All HTTP route handlers that interact with persistent data stores
- API endpoints that perform authentication, authorization, and business logic operations
- Request processing pipelines that include validation, database access, and response generation stages
- Handlers that manage stateful resources including user sessions, verification tokens, and workflow state

Out of scope:
- Static file serving and asset delivery endpoints
- Health check and monitoring endpoints that do not access shared state
- Client-side state management and browser-based concurrency control
- Database-internal concurrency mechanisms and isolation level configuration

Exceptions:
- EXC-001: GET endpoints that perform write operations for legacy compatibility or third-party integration requirements
- EXC-002: High-throughput read endpoints that accept eventual consistency and stale reads for performance optimization

## Rationale

- The evidence shows consistent use of HTTP method-based routing (POST, GET, PATCH, DELETE) across 5 route handlers, with each method corresponding to specific database operation patterns (create, read, update, delete), establishing HTTP methods as the primary concurrency boundary marker
- Database access patterns include complex multi-step operations (findUnique followed by create, multiple findUnique calls with authorization checks) that require transactional consistency to prevent race conditions between validation and execution
- Input validation using schema parsers (safeParse, parse) occurs before database operations in all observed handlers, creating a validation gate that must be enforced consistently to prevent invalid data from reaching the database layer
- The presence of authentication checks, authorization validation, and business logic within route handlers creates a multi-stage pipeline where concurrency boundaries must be maintained across stages to prevent time-of-check-time-of-use vulnerabilities

## Consequences

Positive:
- Explicit HTTP method-based concurrency boundaries make operation semantics clear to clients and enable appropriate caching, retry, and idempotency strategies
- Consistent validation-before-execution patterns reduce the risk of invalid data reaching the database and simplify error handling
- Transactional boundaries around multi-step operations ensure atomicity and prevent partial updates that could leave the system in an inconsistent state
- Authorization checks before database queries prevent unauthorized data access and reduce the attack surface for privilege escalation vulnerabilities

Negative:
- Strict transactional boundaries may increase database lock contention and reduce throughput for high-concurrency workloads
- Mandatory validation before database access adds latency to request processing and may require additional round-trips for complex validation logic
- Pessimistic locking strategies for shared resources can create deadlock risks if not carefully designed and monitored
- Enforcement of HTTP method semantics may require refactoring legacy endpoints that violate REST conventions, creating migration overhead

## Alternatives

- Use application-level locking with distributed lock managers or cache-based mutexes instead of database transactions (rejected)
  Rejected because: Application-level locks introduce additional failure modes (lock service unavailability, network partitions) and do not provide the same ACID guarantees as database transactions. The evidence shows database operations are already the primary state store, making database-level concurrency control more natural.
  When valid: When operations span multiple independent data stores that cannot participate in a single database transaction, or when lock granularity requirements exceed database capabilities
- Implement event sourcing with append-only logs to eliminate update conflicts and enable optimistic concurrency by default (rejected)
  Rejected because: Event sourcing requires fundamental changes to data modeling, query patterns, and application architecture. The evidence shows traditional CRUD operations with direct database queries, making event sourcing a high-risk migration with unclear benefits for the current access patterns.
  When valid: For new systems with complex audit requirements, temporal queries, or domains where event history is a first-class concern
- Rely on database default isolation levels without explicit transaction management in application code (rejected)
  Rejected because: Default isolation levels vary across database systems and may not provide sufficient guarantees for multi-step operations observed in the evidence. Implicit transactions make concurrency boundaries unclear and harder to reason about during code review and debugging.
  When valid: For simple single-query operations where the database default isolation level provides sufficient consistency guarantees

## Risks

- Deadlocks may occur when multiple transactions acquire locks in different orders, particularly in handlers that access multiple related entities
  Mitigation: Establish consistent lock ordering conventions across all route handlers, implement deadlock detection and retry logic, and monitor database deadlock metrics to identify problematic access patterns
  Owner: Engineering team
- Long-running transactions in route handlers may hold database locks for extended periods, blocking other requests and degrading system throughput
  Mitigation: Set transaction timeout limits, move expensive operations (external API calls, file processing) outside transaction boundaries, and implement request timeout monitoring to detect slow handlers
  Owner: Engineering team
- Inconsistent application of concurrency boundaries across route handlers may create subtle race conditions that only manifest under high load
  Mitigation: Implement automated testing that simulates concurrent requests to critical endpoints, establish code review checklists for concurrency patterns, and use static analysis tools to detect missing transaction boundaries
  Owner: Engineering team and QA team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Identify the database client library used in the project and consult its documentation for transaction API patterns. Wrap multi-step database operations in transaction blocks that provide rollback on error and commit on success.
- For route handlers that perform authorization checks, ensure that the authorization logic executes within the same transaction as the subsequent database queries to prevent time-of-check-time-of-use races where permissions change between validation and execution.
- Implement request-scoped transaction management that automatically rolls back on unhandled exceptions and commits on successful response generation. This ensures that error conditions do not leave partial updates in the database.

## Continuation Context


Verify commands:
- Discover the project's test suite location and identify integration tests for API route handlers. Execute tests that verify concurrent request handling and transaction isolation.
- Locate the project's static analysis configuration and run linting rules that detect database operations outside transaction boundaries or validation bypasses.
- Identify the database query logging configuration and enable transaction boundary logging. Review logs for handlers that perform multiple queries without explicit transaction demarcation.

Accept when:
- All route handlers that perform write operations use appropriate HTTP methods (POST, PATCH, DELETE) and no GET handlers modify state
- Integration tests demonstrate that concurrent requests to the same resource produce consistent results without race conditions or lost updates
- Static analysis reports zero violations of transaction boundary rules and validation-before-execution patterns
- Database transaction logs show that multi-step operations within route handlers execute within explicit transaction boundaries with proper rollback on error

## Enforcement

- Verified by: Automated integration tests that simulate concurrent requests and verify transaction isolation guarantees
- Verified by: Static analysis tools that detect database operations outside transaction boundaries
- Verified by: Code review checklists that verify HTTP method semantics, validation ordering, and transaction usage
- Verified by: Runtime monitoring of database transaction metrics including deadlock rates, lock wait times, and rollback frequencies
- Violation handling: CI pipeline fails if static analysis detects validation bypasses or missing transaction boundaries
- Violation handling: Code review blocks merge if route handlers violate HTTP method semantics or lack explicit concurrency control
- Violation handling: Runtime alerts trigger when database deadlock rates or lock contention exceed thresholds, prompting investigation of handler concurrency patterns
- Violation handling: Post-incident reviews for data integrity issues include analysis of transaction boundaries and concurrency control in affected handlers
- Exception process: Developer submits exception request documenting the specific handler, the concurrency requirement that cannot be met with standard patterns, and proposed compensating controls
- Exception process: Technical lead reviews the request for architectural soundness and verifies that alternative approaches have been considered
- Exception process: Security team reviews exception requests that involve authorization checks or sensitive data access to ensure no time-of-check-time-of-use vulnerabilities are introduced
- Exception process: Approved exceptions are documented in code comments with references to the exception approval and must be reviewed annually for continued validity