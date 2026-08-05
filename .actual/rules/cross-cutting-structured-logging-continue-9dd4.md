# Use Metadata Cache Layer for Long-Running Archive Task Progress Tracking: Structured Logging Continue

These rules are ALWAYS ACTIVE for all long-running archive task implementations, progress tracking workflows, and cache layer integrations within the dataroom freeze archive system.

### Rules

- **R-CACHE-001** MUST: Structured logging MUST continue to record operational events with contextual metadata for audit and debugging purposes.
- **R-CACHE-002** MUST: Archive generation tasks triggered through the task orchestration framework MUST use the metadata cache layer for ephemeral progress state.
- **R-CACHE-003** MUST: Progress metadata MUST use task identifiers as cache keys with namespacing to prevent collisions between concurrent archive generation tasks.
- **R-CACHE-004** MUST: Cache write timeouts and circuit breakers MUST be implemented to prevent cache layer latency from blocking archive generation workflow execution.
- **R-CACHE-005** MUST: Progress stage boundaries MUST be defined based on workflow analysis to ensure monotonic progress values that sum to 1.0 across all stages.
- **R-CACHE-006** SHOULD: Progress values SHOULD be calculated based on actual batch sizes and file counts rather than uniform stage increments.
- **R-CACHE-007** SHOULD: Cache write error logging and fallback to database-backed progress table SHOULD be implemented when cache is unavailable.
- **R-CACHE-008** MAY: Short-duration tasks completing within HTTP timeout windows MAY omit progress tracking with architectural review approval.
- **R-CACHE-009** MAY: Workflows with no client-facing progress requirements MAY use structured logging only with product owner sign-off.

### Verify

```bash
# Discover the project's test suite configuration and execute integration tests
# covering archive task progress tracking workflows
find . -name "*test*" -o -name "*spec*" | grep -E "(archive|progress|task)" | head -5

# Locate cache layer health check endpoints in the deployment configuration
grep -r "health" . --include="*.yml" --include="*.yaml" --include="*.json" | grep -i cache | head -5

# Identify monitoring dashboards tracking cache hit rates and write latencies
grep -r "cache.*latency\|cache.*hit" . --include="*.yml" --include="*.yaml" --include="*.json" | head -5

# Verify cache layer connectivity from task execution environment
grep -r "cache.*timeout\|circuit.*breaker" . --include="*.py" --include="*.js" --include="*.go" | head -5
```

**Accept when:**
- Integration tests demonstrate progress metadata updates at all defined workflow stages with correct values and status text
- Cache layer health checks return successful responses with sub-100ms latency from task execution environment
- Monitoring dashboards show cache write success rates above 99.9% for progress metadata operations under normal load
- Code review checklist confirms cache layer updates for new long-running task implementations
- Monitoring alerts are configured on cache write error rates exceeding defined thresholds

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory for archive task implementations. Integration tests MUST pass before merging code that implements long-running archive workflows. Code review MUST verify cache layer progress updates are present for new task implementations.
</enforcement>