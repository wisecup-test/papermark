# Adopt Direct ORM Update Pattern for Dataroom Freeze State Persistence: Nested Relation Projections

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The dataroom freeze archive workflow requires persisting archive metadata (storage location and content hash) after successful archive generation and upload to object storage
- The system uses an ORM-based data access layer that provides type-safe query builders with where clauses, data payloads, and select projections for database operations
- Archive generation is a long-running background task that must atomically update dataroom state with archive artifacts upon completion
- The workflow coordinates multiple external services (object storage, compute functions) and must maintain referential integrity between dataroom records and their freeze artifacts
- Audit and analytics requirements necessitate querying related entities (views, documents, links, agreements) with complex join patterns and temporal filtering based on event timestamps

## Problem Statement

Background archive workflows must persist generated artifacts and their cryptographic hashes to dataroom records in a way that ensures atomicity, type safety, and referential integrity while supporting complex queries across related entities with temporal constraints and nested projections.

## Decision

1. SHOULD: Nested relation projections SHOULD limit selected fields to those required by the workflow to minimize data transfer and serialization overhead

## Policy Block

- SHOULD Nested relation projections SHOULD limit selected fields to those required by the workflow to minimize data transfer and serialization overhead

In scope:
- All dataroom freeze archive state persistence operations
- Queries retrieving dataroom metadata, views, documents, links, and agreement responses for archive generation
- Updates that persist storage references and cryptographic hashes to dataroom records
- Temporal queries filtering entity versions or events by timestamp relative to workflow execution time

Out of scope:
- Object storage upload operations and multipart upload management
- Archive file generation and streaming operations
- Cryptographic hash computation during archive creation
- Background task scheduling and execution infrastructure
- Audit log CSV generation and formatting logic

## Rationale

- The evidence shows direct ORM update calls with where clauses containing both dataroomId and teamId, enforcing multi-tenant isolation at the data access layer and preventing cross-tenant data corruption
- Atomic updates containing both freezeArchiveUrl and freezeArchiveHash ensure that storage location and content verification remain synchronized, preventing scenarios where location is persisted but hash is missing or vice versa
- Complex queries use include directives with nested select projections and orderBy clauses to retrieve related entities (views with documents, links, agreements) in a single database round-trip, reducing latency and maintaining consistency
- Temporal filtering using comparison operators (v.createdAt <= docView.viewedAt) within the query builder ensures accurate version selection based on view timestamps without requiring application-level filtering of large result sets

## Consequences

Positive:
- Type-safe query construction prevents runtime errors from malformed SQL and provides compile-time validation of entity relationships and field names
- Atomic updates with multi-field payloads ensure consistency between related metadata fields (storage location and hash) without requiring explicit transaction management
- Declarative query projections with include and select reduce over-fetching and minimize data transfer between database and application layers
- Multi-tenant isolation enforced at the data access layer through compound where clauses reduces risk of cross-tenant data leakage

Negative:
- ORM abstraction layer introduces dependency on specific query builder APIs and limits ability to use database-specific optimization features or raw SQL for complex queries
- Complex nested projections with multiple include directives can generate inefficient join queries that may require manual optimization or query plan analysis
- Temporal filtering logic embedded in query builders becomes harder to test in isolation compared to pure functions operating on in-memory data structures
- Migration to different ORM implementations or direct SQL requires rewriting all query construction code rather than just connection layer

## Alternatives

- Use raw SQL queries with parameterized statements for all data access operations (rejected)
  Rejected because: Raw SQL eliminates type safety and compile-time validation of entity relationships, increases risk of SQL injection if parameterization is inconsistent, and requires manual mapping between database rows and application domain objects
  When valid: When query complexity exceeds ORM capabilities or when database-specific optimization features are required for performance-critical paths
- Separate archive metadata updates into individual update calls for storage location and hash (rejected)
  Rejected because: Separate updates introduce consistency risk where one update succeeds and the other fails, leaving dataroom record in inconsistent state with location but no hash or vice versa, requiring explicit transaction management
  When valid: Never valid for related metadata fields that must remain synchronized
- Perform separate queries for each related entity and join results in application code (rejected)
  Rejected because: Multiple round-trips to database increase latency, introduce N+1 query problems, and create consistency windows where related entities may change between queries
  When valid: When related entities are retrieved from different data stores or when query result sets are too large to join efficiently in database

## Risks

- Complex nested include directives may generate inefficient join queries with poor execution plans, causing performance degradation as dataroom size grows
  Mitigation: Profile generated SQL queries using database query plan analysis tools, add appropriate indexes on foreign key and timestamp columns used in joins and filters, and consider query result caching for frequently accessed dataroom metadata
  Owner: engineering team
- ORM version upgrades may introduce breaking changes to query builder APIs or alter query generation behavior, requiring extensive regression testing
  Mitigation: Pin ORM dependency to specific version range in dependency manifest, maintain comprehensive integration test suite covering all query patterns, and review ORM changelog for breaking changes before upgrading
  Owner: engineering team
- Temporal filtering logic using comparison operators against timestamps may produce incorrect results if database and application server time zones are not synchronized
  Mitigation: Store all timestamps in UTC timezone, configure database connection to use UTC, and ensure timestamp comparison operations are timezone-aware
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
- When implementing update operations, always include both entity identifier and tenant identifier in where clauses to enforce multi-tenant isolation at the data access layer, preventing accidental cross-tenant updates
- For queries retrieving entity versions or events with temporal constraints, use comparison operators within the query builder's where clause rather than filtering results in application code to push filtering to database layer
- When designing nested projections with include and select directives, explicitly list only required fields rather than fetching entire related entities to minimize data transfer and serialization overhead

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and identify the ORM library and its resolved version from the lock artifact
- Locate the module containing dataroom update operations and verify that where clauses include both entity and tenant identifiers
- Identify query patterns using include or select directives and confirm they specify explicit field projections rather than fetching entire entities
- Find temporal filtering logic and verify comparison operators are applied within query builder methods rather than post-query filtering

Accept when:
- All dataroom update operations include compound where clauses with both entity identifier and tenant identifier
- Archive metadata updates contain both storage location and cryptographic hash in a single atomic data payload
- Queries for related entities use include or select directives with explicit field projections and temporal filters use query builder comparison operators

## Enforcement

- Verified by: Code review verification that all ORM update calls include multi-tenant where clauses
- Verified by: Static analysis rules detecting update operations without compound where clauses
- Verified by: Integration test suite validating that archive metadata updates are atomic and include both location and hash
- Violation handling: Pull requests containing update operations without multi-tenant where clauses are blocked until corrected
- Violation handling: Static analysis violations trigger build failures in continuous integration pipeline
- Violation handling: Runtime monitoring alerts on database queries missing expected tenant identifier filters
- Exception process: Exceptions for single-tenant administrative operations must be documented with explicit justification in code comments
- Exception process: Alternative isolation mechanisms (row-level security, separate schemas) must be reviewed and approved by security team
- Exception process: All exceptions require tracking issue documenting rationale and planned remediation timeline