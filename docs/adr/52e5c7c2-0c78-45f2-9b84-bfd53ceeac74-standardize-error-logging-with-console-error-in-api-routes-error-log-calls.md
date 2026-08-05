# Standardize Error Logging with Console Error in API Routes: Error Log Calls

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- API route handlers across workflow verification, AI chat, dataroom views, and workflow management endpoints require consistent error visibility for operational debugging and security incident response
- The codebase uses Next.js server-side API routes with Prisma database access, Zod input validation, and next-auth authentication, creating multiple failure points that must be observable
- Error logging patterns appear in POST, GET, PATCH, and DELETE endpoints handling sensitive operations including OTP verification, database queries, authentication checks, and team authorization
- Console-based error logging provides immediate visibility in serverless function logs and container stdout streams without requiring additional observability infrastructure

## Problem Statement

API routes handling authentication, database operations, and business logic failures need a consistent mechanism to record error details for debugging, security monitoring, and incident response without exposing sensitive information to clients or requiring complex logging infrastructure setup.

## Decision

1. MUST: Error log calls MUST include the error object as a second parameter to preserve stack traces and error metadata

## Policy Block

- MUST Error log calls MUST include the error object as a second parameter to preserve stack traces and error metadata

In scope:
- All API route handlers in Next.js server-side routes
- Error handling blocks in database query operations
- Authentication and authorization failure paths
- External service integration error handling
- Input validation failure logging

Out of scope:
- Client-side error logging in browser contexts
- Build-time or compile-time error reporting
- Development-only debug logging
- Success case operational logging

Exceptions:
- EXC-001: Error contains sensitive data that cannot be safely redacted automatically
- EXC-002: High-frequency error conditions would create excessive log volume

## Rationale

- The evidence shows consistent console.error usage across 4 API route files handling critical operations including workflow verification, AI chat, dataroom views, and workflow CRUD, indicating an established pattern for error visibility
- API routes integrate with Prisma database queries, Zod validation, and next-auth authentication, creating multiple failure modes that require observable error states for debugging and security monitoring
- Console error logging provides immediate visibility in serverless function logs and container stdout without requiring additional infrastructure, aligning with the deployment model evidenced by @vercel/functions imports
- The pattern supports security incident response by recording error context for authentication failures, authorization checks, and data access violations while keeping sensitive details server-side

## Consequences

Positive:
- Consistent error visibility across all API routes enables faster debugging and incident response
- Server-side error logging prevents exposure of sensitive error details to clients while maintaining operational observability
- Console-based logging integrates seamlessly with serverless function log aggregation and container orchestration platforms
- Standardized error logging pattern reduces cognitive load for developers working across different API endpoints

Negative:
- Console logging lacks structured metadata and query capabilities compared to dedicated observability platforms
- High-volume error conditions may create excessive log output without built-in rate limiting or sampling
- Error messages must be carefully crafted to avoid accidentally logging sensitive data in descriptive context strings
- Console error output may be lost or truncated in environments with limited log retention policies

## Alternatives

- Adopt a structured logging library with automatic metadata enrichment and log level management (rejected)
  Rejected because: Adds dependency overhead and configuration complexity without evidence of current observability gaps requiring structured logging features
  When valid: When log aggregation queries, correlation IDs, or advanced filtering become operational requirements
- Implement centralized error handling middleware that automatically logs all uncaught exceptions (deferred)
  Rejected because: Requires framework-level integration and may not capture context-specific error details visible at the route handler level
  When valid: When error handling patterns stabilize and common error enrichment logic can be extracted
- Use silent error handling with error responses only, no server-side logging (rejected)
  Rejected because: Eliminates operational visibility into failure modes and prevents debugging of production issues without client-side error reports
  When valid: Never valid for production API routes handling authentication or data access

## Risks

- Developers may accidentally log sensitive data in error context strings or by including full request objects
  Mitigation: Establish code review checklist for error logging statements and provide redaction utility functions for common sensitive fields
  Owner: Security team and engineering leads
- Console error logging may create performance overhead in high-throughput error scenarios without rate limiting
  Mitigation: Monitor log volume metrics and implement sampling for known high-frequency error conditions
  Owner: Platform engineering team
- Inconsistent error message formats may hinder log aggregation and automated alerting
  Mitigation: Document error message format conventions and provide linting rules to enforce consistent context string patterns
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
- Wrap all database query operations, external service calls, and authentication checks in try-catch blocks that log errors before returning appropriate HTTP error responses
- Structure error context strings to identify the operation type and resource without including user input or sensitive identifiers that could leak information
- Review existing API route error handlers to ensure consistent error logging patterns and identify any gaps in error visibility coverage

## Continuation Context


Verify commands:
- Discover and execute the project's static analysis or linting configuration to verify error logging patterns in API route handlers
- Locate the project's test suite and run integration tests that verify error logging behavior in failure scenarios
- Inspect the project's API route files to confirm console error calls exist in catch blocks for database operations and authentication failures

Accept when:
- All API route handlers contain console error logging in catch blocks for database operations, authentication failures, and external service errors
- Error log messages include descriptive context strings and error objects without exposing sensitive data
- Code review checklist includes verification of error logging patterns and sensitive data redaction

## Enforcement

- Verified by: Code review process checks for error logging in new API route handlers
- Verified by: Static analysis rules detect missing error logging in catch blocks
- Verified by: Security review validates that error logs do not expose sensitive data
- Violation handling: Pull requests missing error logging in API route error handlers are blocked until logging is added
- Violation handling: Security team reviews any error logs found to contain sensitive data and requires immediate remediation
- Violation handling: Periodic audits identify API routes with inconsistent error logging patterns for remediation
- Exception process: Request exception through security team review with documented justification
- Exception process: Provide alternative observability mechanism if console logging is insufficient
- Exception process: Document exception rationale in code comments and update ADR with amendment