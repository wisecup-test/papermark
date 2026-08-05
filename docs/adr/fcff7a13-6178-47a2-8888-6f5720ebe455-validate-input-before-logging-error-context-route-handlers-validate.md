# Validate Input Before Logging Error Context: Route Handlers Validate

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all API route handlers that perform input validation and error logging.

## Context

- API route handlers receive untrusted input from external clients and must validate it before processing
- Error logging captures diagnostic context including request parameters, identifiers, and error details for debugging
- Input validation failures and database errors are logged to console for operational visibility
- Validation occurs using schema parsers that return structured success or failure results before database access

## Problem Statement

When API route handlers log errors, they may include unvalidated or partially validated input in the logged context, potentially exposing sensitive data, enabling log injection attacks, or recording malformed identifiers that complicate incident response and security auditing.

## Decision

1. MUST: API route handlers MUST validate all external input using schema validation before including any input-derived values in error log statements.

## Policy Block

- MUST API route handlers MUST validate all external input using schema validation before including any input-derived values in error log statements.

In scope:
- API route handlers that accept POST, GET, PATCH, or DELETE requests with path parameters, query parameters, or request bodies
- Error logging statements that include request-derived identifiers or parameters
- Console error logging within try-catch blocks that handle validation or database errors
- Route handlers that perform database queries using request-derived identifiers

Out of scope:
- Internal service-to-service communication where input is pre-validated
- Logging of system-generated identifiers that do not originate from external requests
- Debug logging in non-production environments with explicit developer consent
- Structured logging frameworks that automatically sanitize output

Exceptions:
- EXC-001: Security incident response requires logging raw input for forensic analysis

## Rationale

- The evidence shows consistent pairing of schema validation using safeParse methods with subsequent error logging, indicating awareness of input validation requirements
- Four route handlers demonstrate the pattern of validating identifiers and request bodies before database access and error logging
- Console error logging appears in catch blocks after validation steps, suggesting error context includes validated data
- The pattern prevents log injection vulnerabilities and ensures logged identifiers match expected formats for reliable incident investigation

## Consequences

Positive:
- Prevents log injection attacks where malicious input could corrupt log files or exploit log processing systems
- Ensures logged identifiers conform to expected formats, improving log searchability and incident response efficiency
- Reduces risk of sensitive data exposure through error logs by enforcing validation boundaries
- Creates consistent error logging patterns across API route handlers

Negative:
- Adds validation overhead before logging, potentially delaying error visibility in edge cases
- May obscure root cause analysis if validation strips context needed to diagnose malformed requests
- Requires developers to maintain awareness of validation state when writing error logging code
- Increases code complexity in error handling paths

## Alternatives

- Log all request data without validation and rely on log sanitization at ingestion time (rejected)
  Rejected because: Defers security control to log infrastructure, creating dependency on external sanitization and increasing attack surface during log transport and storage
  When valid: When centralized log infrastructure provides certified sanitization and the organization accepts the risk of unsanitized logs in transit
- Disable error logging of request-derived data entirely and log only static error messages (rejected)
  Rejected because: Eliminates diagnostic context needed for debugging and incident response, making production issues difficult to investigate
  When valid: In highly regulated environments where logging request data violates compliance requirements
- Use structured logging library with automatic field sanitization (deferred)
  Rejected because: Requires adoption of new logging infrastructure and migration of existing console logging statements
  When valid: When the project adopts a structured logging framework that provides built-in sanitization and the team can invest in migration effort

## Risks

- Developers may inadvertently log unvalidated input when adding new error handling code
  Mitigation: Implement linting rules that detect console logging statements in route handlers before validation checkpoints and require code review for all error logging changes
  Owner: Engineering team
- Validation failures may not be logged with sufficient context to diagnose attack patterns
  Mitigation: Establish separate security logging channel for validation failures that records sanitized metadata without echoing invalid input values
  Owner: Security team
- Performance overhead of validation before logging may impact error handling latency
  Mitigation: Cache validation results from request processing phase and reuse them in error handlers to avoid duplicate validation
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
- Establish validation checkpoints at route handler entry points where schema parsing occurs, and ensure all error logging statements after that point only reference validated data structures
- For database errors, log the validated identifier used in the query rather than re-extracting it from the request
- Consider extracting validated identifiers into typed variables immediately after validation to create clear boundaries between validated and unvalidated data in the code

## Continuation Context


Verify commands:
- Discover the project's static analysis configuration and execute the linter to detect console logging statements that reference request parameters before validation
- Locate the project's test suite directory and run integration tests that verify validation failures do not log raw input values
- Identify the repository's code search tooling and scan for error logging patterns that include request-derived data, then manually verify each occurs after validation

Accept when:
- All error logging statements in API route handlers reference only validated identifiers and parameters
- Static analysis reports zero violations of logging-before-validation patterns
- Integration tests confirm validation failures produce logs without echoing invalid input

## Enforcement

- Verified by: Static analysis in continuous integration pipeline scanning for console logging before validation checkpoints
- Verified by: Code review checklist requiring verification that error logs only include validated data
- Verified by: Automated integration tests that inject malformed input and verify logs do not contain raw invalid values
- Violation handling: Build failure if static analysis detects logging before validation
- Violation handling: Code review rejection with required remediation before merge
- Violation handling: Security team notification for violations detected in production code
- Exception process: Developer submits exception request documenting why unvalidated logging is necessary
- Exception process: Security team reviews request and assesses log injection risk
- Exception process: If approved, exception is documented in code comment with ticket reference and expiration date