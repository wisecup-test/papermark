# Use Metadata Cache Layer for Long-Running Archive Task Progress Tracking: Cache Layer Implementations

These rules are ALWAYS ACTIVE for all long-running archive generation tasks, multi-stage workflows requiring client-visible progress tracking, and ephemeral metadata operations with read-heavy access patterns and short lifecycle.

### Rules

- **R-CACHE-001** MAY: Cache layer implementations MAY use time-to-live expiration policies to automatically purge completed task metadata.
- **R-CACHE-002** MUST: Define progress stage boundaries based on workflow analysis to ensure monotonic progress values that sum to 1.0 across all stages.
- **R-CACHE-003** MUST: Use task identifiers as cache keys with namespacing to prevent collisions between concurrent archive generation tasks.
- **R-CACHE-004** MUST: Implement cache write timeouts and circuit breakers to prevent cache layer latency from blocking archive generation workflow execution.
- **R-CACHE-005** MUST: Implement cache write error logging and fallback to database-backed progress table when cache is unavailable.
- **R-CACHE-006** SHOULD: Calculate progress weights based on actual batch sizes and file counts rather than uniform stage increments.
- **R-CACHE-007** SHOULD: Configure appropriate TTL values and memory limits based on expected task concurrency and duration profiles.

### Verify

```bash
# Discover the project's test suite configuration and execute integration tests
# covering archive task progress tracking workflows
find . -name "*test*" -o -name "*spec*" | grep -E "(archive|progress|cache)" | head -20

# Locate cache layer health check endpoints in the deployment configuration
grep -r "health" . --include="*.yml" --include="*.yaml" --include="*.json" | grep -i cache

# Identify monitoring dashboards tracking cache hit rates and write latencies
grep -r "cache.*latency\|cache.*hit" . --include="*.yml" --include="*.yaml" --include="*.json"
```

**Accept when:**
- Integration tests demonstrate progress metadata updates at all defined workflow stages with correct values and status text
- Cache layer health checks return successful responses with sub-100ms latency from task execution environment
- Monitoring dashboards show cache write success rates above 99.9% for progress metadata operations under normal load
- Code review checklist confirms cache layer updates for new long-running task implementations
- Monitoring alerts are configured on cache write error rates exceeding defined thresholds

<enforcement>
Claude Code MUST NOT skip or defer verification. Integration tests MUST pass before merging. Code review MUST confirm cache layer updates for new archive workflows. Monitoring alerts MUST be active in production.
</enforcement>