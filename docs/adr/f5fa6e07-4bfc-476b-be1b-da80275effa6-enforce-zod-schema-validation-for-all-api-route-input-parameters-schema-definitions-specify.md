# Enforce Zod Schema Validation for All API Route Input Parameters: Schema Definitions Specify

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all API route handlers that accept external input via request bodies, query parameters, path parameters, or cookies.

## Context

- API routes accept untrusted input from external clients including request bodies, query parameters, path parameters, and cookies that must be validated before processing
- The codebase uses a schema validation library to define type-safe contracts for all incoming data structures including CUID identifiers, email addresses, arrays with size constraints, and nested objects
- Five route handlers demonstrate consistent application of schema validation with safeParse methods that return success/failure results without throwing exceptions
- Validation failures are detected before database queries execute, preventing invalid data from reaching persistence or business logic layers
- The pattern appears across authentication flows, document viewing, AI chat, dataroom access, and workflow management endpoints

## Problem Statement

API routes that process external input without validation are vulnerable to type confusion, injection attacks, malformed data propagation, and runtime errors. Without a standardized validation approach, each endpoint may implement ad-hoc checks leading to inconsistent security posture and maintenance burden.

## Decision

1. MUST: Schema definitions MUST specify type constraints, format requirements, and boundary conditions for all input fields including string formats, numeric ranges, array size limits, and required versus optional fields

## Policy Block

- MUST Schema definitions MUST specify type constraints, format requirements, and boundary conditions for all input fields including string formats, numeric ranges, array size limits, and required versus optional fields

In scope:
- All HTTP route handlers that accept POST, PATCH, PUT, or DELETE requests with request bodies
- All route handlers that read query parameters from URLs
- All route handlers that extract path parameters from dynamic route segments
- All route handlers that read cookie values for authentication or session management
- Validation of identifiers used in database queries including primary keys and foreign keys

Out of scope:
- Internal function calls between modules within the same trust boundary
- Data already persisted in the database and retrieved through ORM queries
- Environment variables and configuration loaded at application startup
- Static type checking performed by the TypeScript compiler

Exceptions:
- EXC-001: Health check or status endpoints that accept no input parameters
- EXC-002: Internal-only endpoints behind authentication middleware that accept pre-validated session tokens

## Rationale

- The evidence shows 5 route handlers across authentication, document access, AI features, and workflow management all applying schema validation with safeParse methods, indicating an established architectural pattern
- Validation occurs before database queries in all observed cases, preventing invalid data from reaching the persistence layer and reducing attack surface
- The use of safe parsing methods that return results rather than throwing exceptions enables explicit error handling and prevents unhandled exceptions from leaking implementation details
- Consistent validation of CUID identifiers, email formats, array bounds, and nested objects demonstrates defense-in-depth against injection, type confusion, and resource exhaustion attacks

## Consequences

Positive:
- Type-safe input validation prevents malformed data from reaching business logic and database layers
- Explicit validation failures enable clear error responses to clients without exposing internal implementation details
- Reusable schema definitions reduce code duplication and ensure consistent validation rules across endpoints
- Early rejection of invalid requests reduces computational cost and database load from processing malicious or malformed input

Negative:
- Schema definitions add maintenance overhead when API contracts evolve requiring updates to validation rules
- Validation logic executes on every request adding latency before business logic processing begins
- Complex nested schemas may become difficult to maintain and test as API complexity grows
- Overly strict validation rules may reject legitimate edge cases requiring exception handling or schema relaxation

## Alternatives

- Rely on TypeScript static type checking without runtime validation (rejected)
  Rejected because: TypeScript types are erased at runtime and provide no protection against malformed external input from HTTP requests
  When valid: Internal function calls within the same codebase where types are enforced at compile time
- Implement custom validation functions for each endpoint without a validation library (rejected)
  Rejected because: Custom validation leads to inconsistent validation logic, higher maintenance burden, and increased likelihood of security gaps
  When valid: Highly specialized validation requirements not supported by standard validation libraries
- Validate input after database queries to simplify early request handling (rejected)
  Rejected because: Late validation allows invalid data to reach the database layer increasing attack surface and wasting computational resources on malformed requests
  When valid: Never appropriate for untrusted external input

## Risks

- Schema definitions may drift out of sync with actual API behavior if validation rules are not updated when business logic changes
  Mitigation: Implement integration tests that verify validation schemas match API contract specifications and reject invalid inputs
  Owner: Engineering team
- Overly permissive validation rules may allow malicious input to pass validation while overly strict rules may reject legitimate requests
  Mitigation: Conduct security review of validation schemas during code review and monitor validation failure rates in production
  Owner: Security team
- Performance degradation on high-throughput endpoints if complex validation schemas execute expensive operations
  Mitigation: Profile validation performance in load testing and optimize schema definitions or cache validation results where appropriate
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
- Define validation schemas in dedicated schema modules organized by domain or feature area to promote reuse across related endpoints
- Use safe parsing methods that return success or failure results and check the success property before accessing validated data to ensure type safety
- Return appropriate HTTP status codes for validation failures with error messages that describe what validation failed without exposing internal implementation details
- For endpoints that accept arrays, always specify maximum length constraints to prevent resource exhaustion from oversized payloads

## Continuation Context


Verify commands:
- Discover the project's test runner configuration and execute the test suite covering API route validation logic
- Discover the project's static analysis or linting configuration and execute checks that enforce validation patterns
- Discover the project's integration test suite and verify tests exist that submit invalid input to API routes and assert validation failures

Accept when:
- All API route handlers that accept external input include schema validation before processing
- Test suite includes negative test cases that verify validation failures for malformed input
- Static analysis or code review confirms no database queries execute before validation completes

## Enforcement

- Verified by: Code review checklist requiring validation schema presence for all new API routes
- Verified by: Integration tests that submit invalid input and assert appropriate error responses
- Verified by: Static analysis rules that detect database queries before validation checks
- Violation handling: Pull requests without input validation for new API routes are blocked until validation is added
- Violation handling: Security team conducts periodic audits of API routes to identify missing or insufficient validation
- Violation handling: Production monitoring alerts on unexpected validation failure rates indicating schema drift or attack attempts
- Exception process: Request exception through architecture review board with justification for why validation is not required
- Exception process: Security team reviews exception request and assesses risk of accepting unvalidated input
- Exception process: Approved exceptions are documented in API specification with rationale and alternative security controls