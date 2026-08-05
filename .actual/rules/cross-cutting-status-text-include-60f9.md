# Use Metadata Cache Layer for Long-Running Archive Task Progress Tracking: Status Text Include

These rules are ALWAYS ACTIVE for all long-running archive generation tasks and multi-stage workflows requiring client-visible progress tracking through the metadata cache layer.

### Rules

- **R-CACHE-001** SHOULD: Status text SHOULD include batch numbers and totals when processing multiple batches to provide granular visibility into archive generation progress.

### Verify

```bash
# Discover the project's test suite configuration and execute integration tests covering archive task progress tracking workflows
find . -name "*test*" -o -name "*spec*" | grep -E "(archive|progress|task)" | head -5

# Locate cache layer health check endpoints in the deployment configuration and verify connectivity from task execution environment
grep -r "health" . --include="*.yml" --include="*.yaml" --include="*.json" | grep -i cache | head -5

# Identify monitoring dashboards tracking cache hit rates and write latencies for progress metadata operations
grep -r "cache.*latency\|cache.*hit" . --include="*.yml" --include="*.yaml" --include="*.json" | head -5
```

**Accept when:**
- Integration tests demonstrate progress metadata updates at all defined workflow stages with correct batch numbers and status text
- Cache layer health checks return successful responses with sub-100ms latency from task execution environment
- Monitoring dashboards show cache write success rates above 99.9% for progress metadata operations under normal load
- Status text fields in cache metadata include batch identifiers (e.g., "Batch 3 of 10") for multi-batch archive workflows

<enforcement>
Claude Code MUST NOT skip or defer verification. All archive task implementations MUST include cache layer progress updates with batch-aware status text. Code review MUST validate that status text includes batch numbers and totals before merge approval.
</enforcement>