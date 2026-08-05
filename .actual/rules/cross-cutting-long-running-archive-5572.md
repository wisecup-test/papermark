# Use Metadata Cache Layer for Long-Running Archive Task Progress Tracking: Long Running Archive

These rules are ALWAYS ACTIVE for all long-running archive generation tasks and multi-stage workflows requiring client-visible progress tracking through the task orchestration framework.

### Rules

- **R-ARCHIVE-001** MUST: Long-running archive tasks MUST update progress metadata at each major workflow stage using a cache layer with key-value semantics.
- **R-ARCHIVE-002** MUST: Define progress stage boundaries based on workflow analysis to ensure monotonic progress values that sum to 1.0 across all stages.
- **R-ARCHIVE-003** MUST: Use task identifiers as cache keys with namespacing to prevent collisions between concurrent archive generation tasks.
- **R-ARCHIVE-004** MUST: Implement cache write timeouts and circuit breakers to prevent cache layer latency from blocking archive generation workflow execution.
- **R-ARCHIVE-005** SHOULD: Calculate progress weights based on actual batch sizes and file counts rather than uniform stage increments.
- **R-ARCHIVE-006** SHOULD: Implement cache write error logging and fallback to database-backed progress table when cache is unavailable.
- **R-ARCHIVE-007** SHOULD: Configure appropriate TTL values and memory limits based on expected task concurrency and duration profiles.

### Verify

```bash
# Discover the project's test suite configuration and execute integration tests
# covering archive task progress tracking workflows
find . -name "*test*" -o -name "*spec*" | grep -i archive | head -5

# Locate cache layer health check endpoints in the deployment configuration
grep -r "health" . --include="*.yml" --include="*.yaml" --include="*.json" | grep -i cache | head -5

# Identify monitoring dashboards tracking cache hit rates and write latencies
grep -r "cache.*latency\|cache.*hit" . --include="*.yml" --include="*.yaml" --include="*.json" | head -5
```

**Accept when:**
- Integration tests demonstrate progress metadata updates at all defined workflow stages with correct values and status text
- Cache layer health checks return successful responses with sub-100ms latency from task execution environment
- Monitoring dashboards show cache write success rates above 99.9% for progress metadata operations under normal load
- Code review checklist confirms cache layer updates for new long-running task implementations
- Monitoring alerts on cache write error rates are configured and active

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-ARCHIVE rules are mandatory for archive task implementations. Code review MUST block merge if long-running tasks lack cache layer progress updates. CI pipeline MUST fail if integration tests do not verify progress tracking for new archive workflows.
</enforcement>