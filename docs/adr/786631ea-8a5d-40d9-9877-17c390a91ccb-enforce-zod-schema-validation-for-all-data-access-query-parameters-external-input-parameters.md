# Enforce Zod Schema Validation for All Data Access Query Parameters: External Input Parameters

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all data access operations that accept external input parameters used in database queries.

## Context

- The codebase processes external input parameters (CUID identifiers, email addresses, page numbers, arrays) that are directly used in database query predicates and selection criteria
- Five API route handlers demonstrate consistent application of schema validation before database operations, with validation results checked via safeParse patterns
- Database queries use Prisma ORM with findUnique, findMany, create, update, and delete operations that accept user-controlled identifiers in where clauses
- API endpoints handle both authenticated session-based requests and unauthenticated token-based requests, requiring validation across multiple authentication contexts
- The pattern coordinates input validation with access control checks, session management, and multi-step database operations involving related entities

## Problem Statement

Without systematic input validation before database queries, API endpoints risk SQL injection vulnerabilities, type coercion errors, and unauthorized data access through malformed identifiers. The system must ensure all external input used in query predicates is validated against explicit schemas before reaching the data access layer, while maintaining performance and developer ergonomics across diverse endpoint patterns.

## Decision

1. MUST: All external input parameters used in database query predicates, where clauses, or selection criteria MUST be validated against an explicit schema before the query is constructed or executed

## Policy Block

- MUST All external input parameters used in database query predicates, where clauses, or selection criteria MUST be validated against an explicit schema before the query is constructed or executed

In scope:
- All HTTP request handlers that accept query parameters, path parameters, or request body fields used in database operations
- All database query construction code that incorporates external input into where clauses, selection criteria, or data modification operations
- All API endpoints handling both authenticated and unauthenticated requests where input parameters control data access
- All multi-step operations where validated input from one query is used in subsequent queries

Out of scope:
- Internal function parameters derived from already-validated data
- Database queries using only hardcoded constants or server-generated values
- Validation of response data or internal data transformations
- Schema validation for non-database operations such as external API calls or file operations

Exceptions:
- EXC-001: The input parameter is derived from a server-side session or authentication token that has already undergone validation in middleware
- EXC-002: The database query uses a stored procedure or database function that performs its own parameter validation

## Rationale

- The evidence shows consistent application of schema validation with safeParse patterns across five distinct API route handlers, indicating an established architectural pattern for input security
- Database queries consistently use validated CUID identifiers in where clauses, demonstrating that validation prevents type coercion vulnerabilities and ensures referential integrity
- The pattern coordinates validation with access control checks and multi-entity queries, ensuring that malformed input cannot bypass authorization logic or cause cascading query failures
- Using non-throwing safeParse methods allows request handlers to distinguish between validation failures and runtime errors, enabling appropriate HTTP status codes and error messages

## Consequences

Positive:
- Prevents SQL injection and type coercion vulnerabilities by ensuring all query parameters conform to expected types and formats before database execution
- Provides clear error messages to API clients when input validation fails, improving debuggability and API usability
- Enables static type inference from validated schemas, reducing runtime type errors and improving IDE support
- Creates a consistent validation pattern across all API endpoints, reducing cognitive load for developers and simplifying security audits

Negative:
- Adds validation overhead to every request, increasing response latency by the time required for schema parsing and validation
- Requires maintaining parallel schema definitions alongside database models, creating potential for schema drift if not synchronized
- May reject valid edge-case inputs if schemas are overly restrictive, requiring schema updates to accommodate legitimate use cases
- Increases bundle size and memory footprint due to schema definition objects and validation runtime code

## Alternatives

- Rely on ORM type safety and database constraints without explicit input validation (rejected)
  Rejected because: ORM type coercion can mask malformed input and database constraints provide poor error messages to API clients, while validation failures would occur late in the request lifecycle after authorization checks
  When valid: Only in internal services with trusted input sources where performance is critical and database constraints provide sufficient protection
- Perform validation using manual type checking and conditional logic (rejected)
  Rejected because: Manual validation is error-prone, difficult to maintain consistently across endpoints, and lacks the composability and type inference benefits of schema-based validation
  When valid: For simple single-parameter validations where schema library overhead is unjustified
- Validate input in middleware layers before reaching route handlers (deferred)
  Rejected because: Middleware validation requires route-specific schema configuration and may not have access to request context needed for conditional validation logic
  When valid: For common validation patterns shared across many routes, such as authentication token format validation or rate limiting parameter checks

## Risks

- Schema definitions may diverge from database models over time, allowing validated input that violates database constraints or rejecting valid input
  Mitigation: Implement automated tests that verify schema validation accepts all valid database values and rejects invalid ones; consider code generation to derive schemas from database models
  Owner: Engineering team
- Overly permissive validation schemas may allow malicious input patterns that exploit application logic vulnerabilities downstream of validation
  Mitigation: Conduct security review of all validation schemas to ensure they enforce minimum necessary constraints; add custom refinements for business logic validation beyond type checking
  Owner: Security team
- Validation performance overhead may become significant for high-throughput endpoints or complex nested schemas
  Mitigation: Profile validation performance in production; cache compiled schemas; consider moving validation to edge middleware for frequently accessed endpoints
  Owner: Performance engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Define validation schemas as exported constants in dedicated schema modules organized by domain or feature area to promote reuse and maintainability
- For endpoints handling multiple request types, create separate schemas for each request pattern and use discriminated unions or conditional logic to select the appropriate schema
- When validation fails, log the validation error details for debugging but return sanitized error messages to clients that do not expose internal schema structure or validation logic
- For array parameters with length constraints, set maximum bounds based on application performance characteristics and database query optimization limits

## Continuation Context


Verify commands:
- Discover the project's test suite configuration and locate integration tests for API route handlers; execute tests that verify validation rejection behavior for malformed input
- Discover the project's static analysis or linting configuration; run type checking to verify that database query parameters are derived from validated schema types
- Discover the project's code search or grep capabilities; search for database query construction patterns and verify each query using external input has corresponding validation logic

Accept when:
- All API route handlers that accept external input parameters demonstrate schema validation with safeParse or equivalent before database queries
- Test suite includes negative test cases verifying that malformed CUID identifiers, out-of-range array lengths, and invalid types are rejected with appropriate error responses
- Static type checking confirms that all database query where clauses using external input receive parameters typed from validated schemas

## Enforcement

- Verified by: Automated integration tests that verify validation behavior for each API endpoint
- Verified by: Static type checking that ensures query parameters are derived from validated types
- Verified by: Code review checklist requiring validation logic for all new database query code
- Verified by: Security scanning tools that detect database queries using unvalidated external input
- Violation handling: CI pipeline fails if static type checking detects unvalidated parameters in database queries
- Violation handling: Code review blocks merge if validation logic is missing for new API endpoints
- Violation handling: Security scanner findings for unvalidated database queries are treated as high-severity issues requiring immediate remediation
- Violation handling: Runtime monitoring alerts on unexpected database errors that may indicate validation bypass
- Exception process: Developer submits exception request documenting why validation is unnecessary and what alternative protections exist
- Exception process: Security team reviews exception request and assesses risk of validation bypass
- Exception process: If approved, exception is documented in code comments with reference to approval and expiration date
- Exception process: Exceptions are reviewed quarterly and revoked if risk assessment changes