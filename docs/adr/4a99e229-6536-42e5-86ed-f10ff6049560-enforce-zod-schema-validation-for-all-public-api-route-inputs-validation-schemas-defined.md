# Enforce Zod Schema Validation for All Public API Route Inputs: Validation Schemas Defined

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all agent operations involving public API route implementation, modification, or review within the Security & Compliance > Secure Coding Practices domain.

## Context

- The codebase implements public API routes using a server framework that exposes POST, GET, PATCH, and DELETE endpoints handling external requests with untrusted input
- Evidence shows consistent use of schema validation libraries across 5 route handlers processing request bodies, query parameters, and path parameters before business logic execution
- API routes interact with persistent data stores through ORM queries that require type-safe, validated inputs to prevent injection attacks and data integrity violations
- Routes handle sensitive operations including authentication tokens, email verification codes, document access control, and workflow state transitions requiring strict input contracts
- The pattern emerges in security-critical paths including workflow verification, document page access, AI chat creation, dataroom views, and workflow management endpoints

## Problem Statement

Public API routes accepting external HTTP requests must validate all inputs against explicit schemas before processing to prevent injection attacks, type coercion vulnerabilities, and business logic bypasses. Without mandatory validation, routes risk processing malformed data that could compromise data integrity, bypass authorization checks, or trigger unhandled runtime errors in downstream database queries and business logic.

## Decision

1. SHOULD: Validation schemas SHOULD be defined as reusable schema objects separate from route handler implementations to enable testing and reuse

## Policy Block

- SHOULD Validation schemas SHOULD be defined as reusable schema objects separate from route handler implementations to enable testing and reuse

In scope:
- All HTTP route handlers exposed as public API endpoints accepting external requests
- Request body parsing and deserialization logic in POST, PATCH, and PUT endpoints
- Query parameter extraction and processing in GET and DELETE endpoints
- Path parameter extraction used in resource identification and authorization checks
- Input validation preceding database queries, ORM operations, or external service calls

Out of scope:
- Internal function parameters within the same module or service boundary
- Data validation for responses returned to clients
- Schema validation for data read from trusted internal data stores
- Type checking performed by static analysis tools at build time

Exceptions:
- EXC-001: Route handlers implementing health check or status endpoints that accept no input parameters
- EXC-002: Middleware or framework-level validation already enforces identical schema constraints before route handler execution

## Rationale

- Evidence shows 5 route handlers consistently applying schema validation with safeParse methods before database queries, establishing a proven pattern for preventing injection and type coercion attacks
- The pattern validates CUID formats, email strings, numeric ranges, and array bounds matching the constraints required by downstream ORM queries and business logic
- Validation occurs at the API boundary before authorization checks and database access, providing defense-in-depth by rejecting malformed requests early in the request lifecycle
- The 91.76% confidence across multiple security-critical endpoints demonstrates this is an established architectural pattern rather than isolated implementation choices

## Consequences

Positive:
- Prevents injection attacks by validating input types and formats before constructing database queries or executing business logic
- Reduces runtime errors and unhandled exceptions from type mismatches or missing required fields
- Provides clear API contracts through explicit schemas that serve as documentation for consumers
- Enables early rejection of malformed requests with appropriate error responses before consuming resources on invalid operations

Negative:
- Adds implementation overhead requiring schema definition and validation logic for every route handler
- Increases response latency for all requests due to validation processing before business logic execution
- Creates maintenance burden requiring schema updates whenever API contracts change
- May produce verbose error messages requiring careful design to avoid information disclosure vulnerabilities

## Alternatives

- Rely on framework-level type coercion and optional validation middleware without explicit per-route schemas (rejected)
  Rejected because: Framework type coercion often silently converts invalid inputs rather than rejecting them, and optional middleware creates inconsistent validation coverage across routes
  When valid: Only acceptable for internal APIs with trusted clients where input contracts are enforced by shared type definitions
- Implement validation logic using manual type checks and conditional statements within route handlers (rejected)
  Rejected because: Manual validation is error-prone, difficult to test comprehensively, and creates scattered validation logic that is hard to audit for completeness
  When valid: May be appropriate for simple single-field validations where schema libraries would add disproportionate complexity
- Generate validation code automatically from OpenAPI specifications or type definitions (deferred)
  Rejected because: Not rejected but not currently implemented; would require tooling integration and build pipeline changes
  When valid: Valuable for large APIs where maintaining separate schemas and specifications creates synchronization risks

## Risks

- Incomplete validation coverage where some route handlers omit validation or validate only subset of inputs, creating security gaps
  Mitigation: Implement automated detection scanning route handlers for missing validation calls and enforce validation requirements through code review checklists
  Owner: Security team
- Schema definitions diverging from actual business requirements, allowing invalid but schema-compliant inputs to reach business logic
  Mitigation: Establish schema review process tied to API contract changes and maintain integration tests validating end-to-end behavior with boundary inputs
  Owner: Engineering team
- Validation error messages exposing internal implementation details or enabling enumeration attacks
  Mitigation: Define error message templates that provide actionable feedback without revealing database schemas, internal identifiers, or existence of resources
  Owner: Security team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Define validation schemas as exported constants in dedicated schema modules to enable reuse across route handlers and test suites, following the pattern observed in the evidence where schemas are imported from separate validation modules
- Structure route handlers to perform validation as the first operation after extracting request data, returning early with error responses before executing authorization checks or database queries
- For routes accepting multiple input sources, validate each source independently and combine validation results to provide comprehensive error feedback identifying all invalid fields in a single response

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the linting or type-checking command to verify all route handlers include validation calls before business logic
- Locate the project's test suite directory structure and execute the integration test command targeting API route handlers to verify validation behavior with invalid inputs
- Identify the project's code search or grep capability and scan route handler implementations for validation method invocations to confirm coverage across all public endpoints

Accept when:
- All public API route handlers contain schema validation calls processing request inputs before database queries or business logic execution
- Integration tests demonstrate that routes return appropriate HTTP error responses when provided with invalid inputs violating schema constraints
- Code review or automated scanning confirms no route handlers access request body properties, query parameters, or path parameters without prior validation

## Enforcement

- Verified by: Automated static analysis scanning route handler implementations for validation method calls before business logic execution
- Verified by: Code review checklist requiring validation coverage verification for all new or modified API routes
- Verified by: Integration test suite coverage reports confirming validation error paths are exercised for each route
- Violation handling: Pull requests adding or modifying route handlers without validation are blocked by automated checks and require remediation before merge
- Violation handling: Security team conducts periodic audits of route handlers and files remediation tickets for missing validation with priority based on endpoint sensitivity
- Violation handling: Runtime monitoring alerts on unhandled exceptions or type errors in route handlers indicating potential validation gaps
- Exception process: Exception requests must document the specific route, justify why validation is unnecessary, and identify alternative security controls
- Exception process: Security team reviews exception requests and approves only when alternative controls provide equivalent protection
- Exception process: Approved exceptions are documented in code comments referencing the exception approval and are reviewed annually for continued validity