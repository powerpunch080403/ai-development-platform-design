# ADR-0026 WorkerRun Lease, Stale Recovery, and Retry Policy

## Status

Proposed

## Date

2026-06-25

## Context

ADR-0025 established Worker identity, runtime adapters, capacity, queue ordering, and durable WorkerRun state. It also introduced explicit queued/running/terminal state guidance for TaskAttempt and WorkerRun.

The implementation repository now reflects that direction:

- `RecordStatus` includes `queued`, `running`, `succeeded`, `failed`, `cancelled`, and `skipped`.
- `TaskAttemptStatus` includes `queued_worker`, `running_worker`, `worker_failed`, and terminal review/result states.
- `TaskAttempt` already has `lease_expires_at`, but `WorkerRun` does not yet have explicit lease or heartbeat fields.
- WorkerRun creation marks the WorkerRun as `queued` and the TaskAttempt as `queued_worker`.
- Worker capacity checks only treat `WorkerRun.running` as active capacity.
- Queue drain selects `queued` WorkerRuns and skips a Task if another WorkerRun for that Task is already `running`.
- The AGY background runner executes WorkerRuns serially and auto-drains the queue after terminal completion.
- The subprocess runtime enforces process-level timeouts and writes ProcessRun terminal state, but there is no scheduler-level stale-running recovery for a WorkerRun that remains `running` after process/runtime interruption.

This leaves one important long-term failure mode: a WorkerRun can remain `running` even when the underlying worker process, background task, or runtime boundary is no longer alive. If that happens, the Worker capacity guard continues to see an active WorkerRun and queued work can be blocked indefinitely.

Open-source systems use similar safeguards:

- Apache Airflow models `queued`, `running`, `success`, `failed`, and `up_for_retry` as distinct Task Instance states, and documents that TaskInstances can get stuck in `running` when the associated worker/job is inactive. Airflow periodically finds these, cleans them up, and marks the TaskInstance failed or retries it if retries remain.
- Temporal recommends idempotent Activities, treats heartbeats as part of failure detection, and distinguishes timeout categories including heartbeat timeouts. Temporal also notes that failed Activity attempts restart from the initial state unless checkpointing is explicitly done through heartbeat details.
- Kubernetes uses Lease objects for heartbeats and coordination. Node heartbeats update `spec.renewTime`, and the control plane uses that timestamp to determine node availability. Kubernetes Jobs also support `activeDeadlineSeconds`; when the deadline is reached, running Pods are terminated and the Job becomes `Failed` with `DeadlineExceeded`. Kubernetes documents that this is a permanent Job failure requiring manual intervention.
- Celery warns that tasks should ideally be idempotent because the worker cannot know whether re-execution is safe. It acknowledges already-started tasks by default so they are not automatically re-executed; it also warns that indefinitely blocking tasks can stop a worker from doing other work and recommends explicit timeouts.

The platform's WorkerRuns are not simple pure functions. They can create worktrees, mutate files, create result commits, and write audit/artifact records. Therefore automatic retry is more dangerous here than in simple message-processing systems unless retry safety is explicitly declared.

## Decision

Adopt a staged WorkerRun liveness model:

```text
Phase 1: stale-running recovery primitive
Phase 2: heartbeat-backed WorkerRun lease
Phase 3: scheduler/periodic recovery
Phase 4: explicit retry policy for safe tasks only
```

### Phase 1: stale-running recovery primitive

The first implementation step should be a manual or system-triggered recovery primitive, not a full scheduler.

A WorkerRun is stale when:

```text
WorkerRun.status == running
AND liveness reference time is older than configured timeout
```

Initial liveness reference should use the best available timestamp in this order:

```text
WorkerRun.updated_at
WorkerRun.started_at
WorkerRun.created_at
```

When a WorkerRun is stale:

```text
WorkerRun.status = failed
WorkerRun.failed_at = now
WorkerRun.error_code = "STALE_WORKER_RUN"
WorkerRun.error_message = "WorkerRun stale timeout exceeded"

Linked TaskAttempt.status = worker_failed
Linked TaskAttempt.failed_at = now
Linked TaskAttempt.error_code = "STALE_WORKER_RUN"
Linked TaskAttempt.error_message = "WorkerRun stale timeout exceeded"
```

The recovery must also record an audit event containing:

```text
worker_run_id
task_attempt_id
worker_id
timeout_seconds
last_liveness_at
recovery_trigger
```

Worktree, ProcessRun, artifacts, logs, and partial files must be preserved. No cleanup, branch deletion, or automatic retry is allowed in Phase 1.

Queue behavior after recovery:

```text
stale running WorkerRun -> failed
capacity guard no longer sees it as active
next queued WorkerRun can be drained normally
```

### Phase 2: heartbeat-backed WorkerRun lease

After Phase 1 is stable, WorkerRun should gain explicit liveness fields rather than relying only on `updated_at`:

```text
WorkerRun.last_heartbeat_at
WorkerRun.lease_expires_at
WorkerRun.heartbeat_source
```

TaskAttempt already has `lease_expires_at`; this field should either be used as the attempt-level aggregate lease or replaced by a clearer attempt/worker-run split in a later migration.

The Worker execution boundary should heartbeat during long-running execution. For local CLI/process execution, a heartbeat can initially be emitted by the platform runner while waiting for the subprocess. Later, adapter-specific heartbeats may be added if the external runtime exposes progress signals.

Lease expiration should be treated as a liveness failure, not as a signal to automatically retry.

### Phase 3: scheduler/periodic recovery

Once the primitive is proven safe, add a periodic recovery loop or scheduler-triggered System ToolCall.

The scheduler must:

- run at a conservative interval;
- acquire a scheduler lease or idempotency key so multiple scheduler instances do not race;
- update records transactionally;
- emit audit events;
- never mutate source repositories or delete worktrees during recovery;
- run queue drain after recovery only through the normal queue boundary.

### Phase 4: explicit retry policy

Automatic retry is not the default.

Retry may be introduced only when the task/attempt declares retry safety:

```text
retry_policy.enabled = true
retry_policy.max_attempts
retry_policy.backoff
retry_policy.retryable_error_codes
retry_policy.requires_idempotent_worker = true
```

For code-changing WorkerRuns, the default is no automatic retry. The Owner may create a follow-up Attempt after inspecting the failed Attempt, worktree, logs, and artifacts.

If retry is eventually allowed, it must create a new Attempt or new WorkerRun record according to the policy. It must not overwrite the failed WorkerRun or erase its evidence.

## Alternatives Considered

### Leave stale WorkerRuns running until manual database repair

Rejected. This preserves raw state but can permanently block Worker capacity and queued work.

### Mark stale TaskAttempt as generic `failed`

Rejected for the default path. The failure is specifically a Worker execution/liveness failure, not necessarily a task-content failure. `worker_failed` preserves a clearer cause for Owner review and later retry policy.

### Automatically create a follow-up Attempt when stale is detected

Rejected for now. Code-changing workers are not guaranteed idempotent, and automatic follow-up can create duplicate work, conflicting diffs, or misleading task history.

### Automatically re-run the same WorkerRun

Rejected. Re-running the same record would destroy audit clarity and make it difficult to distinguish the original stale execution from the recovery execution.

### Build the full heartbeat scheduler immediately

Rejected for the next implementation slice. The safer sequence is to first implement the stale recovery primitive, dogfood it, then add heartbeat fields and scheduler automation.

## Consequences

Positive:

- Prevents Worker capacity from being permanently blocked by stale `running` records.
- Preserves all worktree/log/artifact evidence for diagnosis.
- Keeps retry decisions under Owner control until retry safety is explicit.
- Aligns with ADR-0025's durable queue and execution state model.
- Creates a clean path toward heartbeat-backed leases and scheduler recovery.

Negative:

- Phase 1 requires manual/system-triggered recovery rather than fully automatic recovery.
- Some stale runs may remain blocked until recovery is invoked.
- Without WorkerRun heartbeat fields, Phase 1 must rely on `updated_at`/`started_at` heuristics.
- Automatic retry remains deferred even when a failure might be transient.

## Implementation Guidance

Recommended next implementation slice:

```text
PR #16: stale-running recovery primitive
```

Minimum scope:

- Add config such as `worker_run_stale_timeout_seconds`.
- Add a recovery function that finds stale `WorkerRun.running` records.
- Mark WorkerRun as `failed` and linked TaskAttempt as `worker_failed`.
- Preserve worktree, artifacts, ProcessRun records, and logs.
- Add audit event `worker_run.stale_recovered` or equivalent.
- Add tests for:
  - stale WorkerRun recovery;
  - non-stale WorkerRun not recovered;
  - linked TaskAttempt becomes `worker_failed`;
  - queue capacity unblocks after recovery;
  - no automatic retry/new Attempt is created.

Out of scope for PR #16:

- WorkerRun heartbeat columns.
- Scheduler loop / heartbeat daemon.
- automatic retry.
- automatic follow-up Attempt creation.
- worktree cleanup.
- source repository mutation.

## Open-source References

- Apache Airflow Task Instance lifecycle and heartbeat timeout: https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html
- Temporal Activities and heartbeat/timeouts: https://docs.temporal.io/activities and https://docs.temporal.io/develop/go/best-practices/error-handling
- Kubernetes Leases and node heartbeat model: https://kubernetes.io/docs/concepts/architecture/leases/
- Kubernetes Job `activeDeadlineSeconds`: https://kubernetes.io/docs/concepts/workloads/controllers/job/
- Celery task idempotency, late acknowledgement, worker-lost behavior, and time limits: https://docs.celeryq.dev/en/stable/userguide/tasks.html

## Revisit When

- WorkerRun gains `last_heartbeat_at` and `lease_expires_at` fields.
- The platform adds a scheduler or periodic System ToolCall runner.
- Worker adapters can report adapter-native progress or heartbeat signals.
- Retry policy is added to Task/TaskAttempt/WorkerRun.
- Multiple local CLI Workers or higher `max_parallel` configurations are supported.
