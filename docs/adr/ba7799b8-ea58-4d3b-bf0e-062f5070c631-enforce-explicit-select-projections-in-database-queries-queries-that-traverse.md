# Enforce Explicit Select Projections in Database Queries: Queries That Traverse

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- API routes handling sensitive operations require controlled data exposure to minimize information leakage and reduce payload size
- Database queries across authentication, authorization, and multi-tenant boundaries must prevent accidental disclosure of sensitive fields
- Input validation using schema parsers precedes database access, establishing a defense-in-depth pattern where validation and projection work together
- Public API contracts expose data to external viewers, authenticated users, and workflow participants with varying permission levels
- Query patterns consistently use explicit select clauses with nested relation projections rather than retrieving entire entity graphs

## Problem Statement

Without explicit field projection in database queries, API routes risk exposing sensitive data through over-fetching, increase network payload sizes unnecessarily, and create implicit coupling between database schema evolution and API contracts. The absence of select clauses makes it difficult to audit what data flows through each endpoint and complicates compliance with data minimization principles.

## Decision

1. MUST: Queries that traverse relations must specify select clauses for nested entities to prevent transitive over-fetching

## Policy Block

- MUST Queries that traverse relations must specify select clauses for nested entities to prevent transitive over-fetching

In scope:
- API route handlers that return data to external clients or viewers
- Database queries that retrieve user data, team data, or multi-tenant resources
- Operations that cross authentication or authorization boundaries
- Queries that access entities containing email addresses, tokens, passwords, or configuration data
- Public endpoints that accept link identifiers or preview tokens

Out of scope:
- Internal background jobs that process data without external exposure
- Database migrations and schema introspection operations
- Development and testing utilities that require full entity inspection

Exceptions:
- EXC-001: Administrative endpoints require full entity retrieval for debugging or support operations
- EXC-002: Internal service-to-service calls within the same trust boundary require complete entity graphs for performance optimization

## Rationale

- Evidence shows 5 files with consistent explicit select patterns across authentication flows, document access, AI chat authorization, dataroom views, and workflow management, indicating an established architectural practice
- Select clauses consistently project only fields required for authorization decisions, response construction, or business logic, demonstrating intentional data minimization
- Nested select clauses on relations prevent transitive over-fetching while maintaining query expressiveness for complex authorization checks
- The pattern appears alongside input validation using schema parsers, forming a defense-in-depth approach where both input and output data are explicitly controlled

## Consequences

Positive:
- Reduces risk of accidental sensitive data exposure through API responses
- Decreases network payload sizes and database query execution time by retrieving only necessary fields
- Makes data access patterns explicit and auditable through code inspection
- Decouples API contracts from database schema evolution, allowing schema changes without breaking clients
- Facilitates compliance with data minimization principles required by privacy regulations

Negative:
- Increases verbosity of database query code, requiring explicit enumeration of fields
- Creates maintenance burden when new fields need to be added to multiple query sites
- May lead to inconsistent projections across similar queries if not carefully reviewed
- Requires developers to understand the full data flow to determine the minimal field set

## Alternatives

- Retrieve full entities and filter sensitive fields in a post-processing layer before serialization (rejected)
  Rejected because: Post-processing filtering increases memory usage, network overhead between database and application, and creates a second location where data access policy must be maintained, increasing the risk of inconsistency
  When valid: May be acceptable for internal administrative tools where performance is not critical and centralized filtering logic provides value
- Define reusable projection objects or view models that encapsulate common field sets (deferred)
  Rejected because: While this reduces duplication, it abstracts the data access pattern away from the query site, making it harder to audit what data each endpoint accesses. The current pattern prioritizes visibility over reusability
  When valid: Appropriate when the same projection is used across many endpoints and the projection is stable, with clear naming that indicates its security implications
- Use database views or row-level security policies to enforce field-level access control (rejected)
  Rejected because: Database-level enforcement requires maintaining security logic in two places and may not provide the flexibility needed for context-dependent projections based on request parameters or user roles
  When valid: Valuable as a defense-in-depth layer for highly sensitive data, but should complement rather than replace application-level projections

## Risks

- Developers may inadvertently omit sensitive fields from select clauses but still expose them through other code paths or serialization logic
  Mitigation: Implement automated tests that verify API responses do not contain sensitive fields, and conduct security-focused code reviews for all data access patterns
  Owner: Security team and engineering team
- Inconsistent application of select clauses across the codebase may create security gaps where some endpoints properly project fields while others do not
  Mitigation: Establish linting rules or static analysis checks that flag database queries without select clauses in public API routes, and maintain a catalog of sensitive fields that must never be exposed
  Owner: Engineering team
- Performance optimization through field projection may be undermined if the database client library or query planner does not efficiently handle selective field retrieval
  Mitigation: Profile query performance with and without select clauses, and verify that the database client library properly translates projections into efficient queries
  Owner: Engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When implementing new API routes, define the select clause before writing the query by listing the fields required for the response contract and authorization logic
- For queries with nested relations, apply the same projection discipline recursively, explicitly selecting fields from each related entity
- Document the rationale for including each field in the select clause when the necessity is not immediately obvious from the surrounding code
- When adding new fields to entities, audit all existing queries to determine whether the new field should be included in their projections

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the linter or type checker to identify database queries without explicit select clauses
- Locate the project's test suite and run integration tests that verify API responses do not contain fields marked as sensitive in the schema
- Identify the project's code review checklist and confirm it includes verification that new database queries in public API routes include explicit select projections

Accept when:
- All database queries in public API routes that retrieve entities with sensitive fields include explicit select clauses
- Static analysis or linting passes without warnings about missing select clauses in security-sensitive contexts
- Integration tests confirm that API responses contain only the fields specified in the select projections and do not leak sensitive data

## Enforcement

- Verified by: Code review process that specifically checks for explicit select clauses in database queries within API routes
- Verified by: Static analysis or linting rules that flag queries without select clauses in files matching public API route patterns
- Verified by: Integration tests that assert API responses do not contain sensitive fields
- Violation handling: Code review blocks merge until select clauses are added to queries in public API routes
- Violation handling: Static analysis failures in continuous integration prevent deployment
- Violation handling: Security team conducts periodic audits of data access patterns and files remediation tickets for violations
- Exception process: Developer documents the rationale for omitting select clause in code comments and pull request description
- Exception process: Security team reviews the exception request and assesses the risk of data exposure
- Exception process: If approved, the exception is recorded in a security audit log with a review date for re-evaluation