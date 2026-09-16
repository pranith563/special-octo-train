Investigate job_2459d611076c489ebf7c218f in this deployed environment. Collect evidence read-only; do not deploy, restart services, cancel/requeue jobs, modify configuration/database records, or delete artifacts.

We have implemented local fixes, but the original four-day stall is not conclusively diagnosed. We need actual data to distinguish budget reconciliation from checkpoint/database or sandbox delays.

1. Identify the environment
- Exact deployed Git SHA and sandbox backend.
- Job status and timestamps, sweep phase, cancellation timestamp, and recorded infrastructure retry/interruption details. Inspect the schema for correct field names; avoid dumping full request/state blobs.
- Use UTC throughout. Distinguish current observations from historical incident evidence.

2. Run these read-only queries. Save the first result as CSV and the second as full JSON, preserving NULLs.

BEGIN TRANSACTION READ ONLY;
SET LOCAL statement_timeout = '30s';

SELECT group_index, group_key, status, measurement_id, created_at, updated_at,
       telemetry ->> 'artifact_logical_bytes' AS artifact_logical_bytes,
       telemetry ->> 'artifact_allocated_bytes' AS artifact_allocated_bytes,
       telemetry ->> 'budget_cases' AS budget_cases,
       telemetry ->> 'budget_bytes' AS budget_bytes,
       telemetry ->> 'producer_blocked_seconds' AS producer_blocked_seconds
FROM public_api.patch_search_conversion_groups
WHERE job_id = 'job_2459d611076c489ebf7c218f'
  AND group_index BETWEEN 2100 AND 2320
ORDER BY group_index;

SELECT group_index, telemetry
FROM public_api.patch_search_conversion_groups
WHERE job_id = 'job_2459d611076c489ebf7c218f'
  AND group_index = 2175;

ROLLBACK;

Also report conversion-group counts by status and the total durable measurement count for this job.

3. Collect effective configuration values
- backlog_max_chunks, backlog_max_bytes, profile_chunk_size
- max_artifact_bytes, max_export_chunk_bytes, min_free_disk_bytes
- conversion_timeout_sec, conversion_max_retries
- infra_resume_max_retries, infra_resume_backoff_sec
- device_keepalive_sec, pipeline_stall_timeout_sec if present
- Public API Postgres pool acquisition and command/query timeouts

Identify the configuration source and whether these values are known to match the incident-time deployment. Do not expose connection URLs, passwords, tokens, or environment dumps.

4. Retrieve relevant worker, sandbox, and Postgres log excerpts
UTC windows:
- 2026-09-11 13:30–14:05
- 2026-09-15 06:30–06:36

Focus on this job, g002175 and its measurement ID, inspection/checkpoint operations, timeouts, connection failures, lock waits, claim/permit loss, process restarts, cancellation, and disk/OOM errors. Preserve timestamps and error details. If logs expired or are inaccessible, state that explicitly.

5. Check storage
Report current free bytes and inodes on the actual sandbox-workspace filesystem. Include historical disk-pressure/OOM evidence if available; current free space cannot establish incident-time capacity.

Return:
- The compact CSV and g002175 JSON.
- Configuration values and timestamped log excerpts.
- A short timeline separating facts from hypotheses.
- Missing/unavailable evidence.

Interpretation guardrails:
- High-water counters are historical maxima, not current occupancy.
- budget_cases/budget_bytes are snapshots taken before checkpointing and reconciliation.
- producer_blocked_seconds counts completed reservation waits.
- If g002175's allocated bytes were at or below its actual reservation (536870912 bytes under the reported defaults), growth in reconcile_bytes could not have blocked that group. This rules out that mechanism; it does not by itself prove a four-day database transaction.
