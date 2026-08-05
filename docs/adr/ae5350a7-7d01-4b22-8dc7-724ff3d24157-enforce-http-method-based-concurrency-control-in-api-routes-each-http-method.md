# Enforce HTTP Method-Based Concurrency Control in API Routes: Each Http Method

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all API route handlers that process HTTP requests and perform database operations.

## Context

- API routes handle multiple HTTP methods (GET, POST, PATCH, DELETE) with distinct authorization, validation, and data access patterns per method
- Each route performs database queries using an ORM client with method-specific selection, filtering, and mutation operations
- Input validation schemas are applied per HTTP method using schema validation libraries with type-safe parsing
- Authentication and authorization checks vary by HTTP method, with some methods requiring team membership verification and others supporting viewer-based access
- Error handling and logging patterns are consistent across routes, with structured error responses and console-based error logging

## Problem Statement

API routes that handle multiple HTTP methods without explicit concurrency control and method-specific validation create security vulnerabilities where unauthorized operations may bypass access controls, input validation may be inconsistent across methods, and concurrent requests may produce race conditions in database state mutations.

## Decision

1. MUST: Each HTTP method handler MUST apply method-specific input validation using schema validation before accessing request body or query parameters

## Policy Block

- MUST Each HTTP method handler MUST apply method-specific input validation using schema validation before accessing request body or query parameters

In scope:
- All HTTP API route handlers that accept external requests
- Database query and mutation operations performed within route handlers
- Input validation and schema parsing for request bodies and query parameters
- Authentication and authorization checks for protected resources
- Error handling and response formatting in API routes

Out of scope:
- Internal service-to-service communication that does not traverse HTTP boundaries
- Background job processors that operate outside the request-response cycle
- Database migrations and schema management operations
- Static file serving and asset delivery

Exceptions:
- EXC-001: Public endpoints that intentionally allow unauthenticated access for specific business requirements
- EXC-002: Legacy routes undergoing incremental migration to the standard pattern

## Rationale

- Evidence shows 5 route files implementing consistent patterns of HTTP method-based request handling with method-specific validation schemas and database access patterns
- The pattern coordinates authentication, input validation, and database operations within a single request handler, isolating concerns by HTTP method while maintaining consistent error handling
- Schema validation libraries provide type-safe parsing with explicit success/failure handling, preventing invalid data from reaching database operations
- Database queries consistently use explicit field selection and WHERE clause filtering to enforce authorization boundaries at the data access layer

## Consequences

Positive:
- Method-specific validation and authorization reduce the attack surface by ensuring each operation type has appropriate security controls
- Explicit field selection in database queries minimizes data exposure and improves query performance
- Consistent error handling patterns across routes improve observability and debugging
- Type-safe schema validation catches malformed input before it reaches business logic or database layers

Negative:
- Each HTTP method requires separate validation schema definitions, increasing code volume and maintenance burden
- Authorization logic may be duplicated across multiple method handlers within the same route file
- Sequential database queries for existence checks followed by mutations create potential race conditions under concurrent access
- Console-based error logging may not provide sufficient structure for production monitoring and alerting

## Alternatives

- Use middleware-based validation and authorization that applies uniformly to all HTTP methods (rejected)
  Rejected because: Evidence shows method-specific validation schemas and authorization requirements that cannot be uniformly applied across all methods without losing security granularity
  When valid: Valid for routes where all HTTP methods share identical validation and authorization requirements
- Implement optimistic locking with version fields to handle concurrent mutations (deferred)
  Rejected because: Not observed in current evidence; would require schema changes and additional complexity
  When valid: Valid when concurrent modification conflicts are frequent and must be detected rather than prevented
- Separate each HTTP method into distinct route files with dedicated handlers (rejected)
  Rejected because: Current pattern groups related operations by resource, which improves code organization and reduces duplication of shared logic
  When valid: Valid for very large route handlers where method-specific logic exceeds several hundred lines

## Risks

- Race conditions between existence checks and subsequent mutations may allow duplicate records or unauthorized state changes under concurrent access
  Mitigation: Use database-level constraints and atomic upsert operations where supported; implement request-level locking for critical operations
  Owner: Engineering team
- Inconsistent application of authorization checks across HTTP methods may create privilege escalation vulnerabilities
  Mitigation: Implement automated security testing that verifies authorization enforcement for each HTTP method; conduct regular security audits of route handlers
  Owner: Security team
- Schema validation failures may expose internal structure through error messages
  Mitigation: Sanitize validation error messages before returning to clients; log detailed errors internally while returning generic messages externally
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
- Organize route handlers by grouping HTTP method handlers within a single route file, with early method validation and explicit error responses for unsupported methods
- Define validation schemas adjacent to their corresponding HTTP method handlers to maintain clear association between validation rules and handler logic
- Structure database queries to include authorization predicates in WHERE clauses rather than performing separate authorization checks after data retrieval
- Implement consistent error response formatting across all routes using a shared error handler that maps validation failures, authorization denials, and database errors to appropriate HTTP status codes

## Continuation Context


Verify commands:
- Discover the project's test suite location and execute integration tests that verify HTTP method handling, input validation, and authorization for API routes
- Discover the project's static analysis configuration and execute type checking to verify schema validation is applied before database operations
- Discover the project's security testing tools and execute authorization tests that attempt unauthorized access across all HTTP methods

Accept when:
- All API route handlers explicitly validate HTTP methods and return appropriate error responses for unsupported methods
- Schema validation is applied to all request inputs before database operations are performed, with validation failures returning structured error responses
- Authorization checks are present for all database mutations and verify ownership or team membership in query predicates
- Integration tests demonstrate that unauthorized requests are rejected with appropriate HTTP status codes for each HTTP method

## Enforcement

- Verified by: Automated integration tests that verify method-specific validation and authorization for each route
- Verified by: Static analysis tools that detect database queries without explicit field selection or authorization predicates
- Verified by: Code review checklist items that verify HTTP method handling, input validation, and authorization patterns
- Verified by: Security scanning tools that test for authorization bypass and input validation vulnerabilities
- Violation handling: CI pipeline failures block merging of code that lacks required validation or authorization checks
- Violation handling: Security scanner findings trigger immediate review and remediation for authorization bypass vulnerabilities
- Violation handling: Code review rejections require addition of missing validation schemas or authorization predicates before approval
- Violation handling: Production monitoring alerts on unexpected HTTP method usage or validation failure patterns
- Exception process: Submit exception request to security team with justification for deviation from standard pattern
- Exception process: Document approved exceptions in route handler comments with reference to approval ticket
- Exception process: Schedule periodic review of exceptions to determine if they can be migrated to standard pattern
- Exception process: Maintain exception registry with expiration dates and responsible owners