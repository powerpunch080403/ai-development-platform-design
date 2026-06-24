# Platform Runtime Reliability Review - Pass 1

## Status

Draft review for design follow-up.

## Date

2026-06-25

## Scope

This is the first broad pass over the implementation repository after PR #15. It is wider than ADR-0026, but it is still a first pass, not a final architecture audit.

Reviewed areas:

- FastAPI route surface and app composition.
- Auth, session, device lifecycle.
- Project and repository registration.
- Conversation, Message, AgentRun, AgentRunStep lifecycle.
- Tool registry, ToolCall envelope, policy, approval, audit.
- WorkItem, Task, TaskAttempt, Worker, WorkerRun lifecycle.
- Owner ToolCall bridge, queue, drain, capacity guard.
- WorkerExecutionService, local background handoff, AGY auto-drain.
- ProcessRun and ProcessRuntimeProvider.
- Worktree creation, result commit, cleanup safety, artifacts.
- Manual worker, mock worker, external CLI and AGY adapter paths.
- Review, approval, and squash merge flow.
- Web API client and shared TypeScript contracts.
- Related design docs: ADR-0017, ADR-0023, ADR-0025, Personal Mode MVP Roadmap.

Not fully reviewed in this pass:

- Every frontend component.
- Every historical migration edge case.
- Full UI and UX completeness.
- Packaging, desktop shell, OS keychain, installer, update flows.
- Team Mode and remote runner design.

## Open-source reference model

The common long-term pattern across Airflow, Temporal, Kubernetes, Celery, and LangGraph is:

```text
durable state machine
+ explicit queued/running/terminal states
+ heartbeat or lease liveness
+ timeout/deadline handling
+ evidence/log preservation
+ explicit retry policy only when safe
```

Mapped to this platform:

- Task, TaskAttempt, WorkerRun, ToolCall, AuditEvent, and ArtifactRef remain product-owned source of truth.
- External workflow or task systems may later live behind service boundaries, not replace the product model.
- Automatic retry is unsafe by default because WorkerRuns can mutate worktrees, create commits, and produce artifacts.
- Owner review and explicit follow-up remain the default recovery path for code-changing Attempts.

## High-priority findings

### P0-1: queued-state transition consistency

After PR #15, enums include `RecordStatus.QUEUED` and `TaskAttemptStatus.QUEUED_WORKER`, but `state_transitions.py` should be updated to make those states first-class.

Expected direction:

```text
TaskAttempt.created -> queued_worker
TaskAttempt.queued_worker -> running_worker | worker_failed | failed | cancelled
WorkerRun.created -> queued | running | cancelled | skipped
WorkerRun.queued -> running | cancelled | skipped | failed
WorkerRun.running -> succeeded | failed | cancelled
```

### P0-2: stale-running recovery primitive

ADR-0026 covers this. The next implementation slice should recover stale `WorkerRun.running` records, mark the WorkerRun failed, mark the linked TaskAttempt as `worker_failed`, preserve evidence, emit audit, and avoid automatic retry or automatic new Attempt creation.

### P0-3: WorkerRun failure semantic normalization

Worker execution and liveness failures should generally map linked TaskAttempt to `worker_failed`, not generic `failed`. Generic `failed` should be reserved for broader system/data-integrity failures.

### P1-1: consolidate Worker execution paths

There are currently two overlapping models:

```text
Owner ToolCall -> queued WorkerRun -> drain -> run
Worker HTTP API -> claim TaskAttempt lease -> heartbeat -> release
```

Long-term target:

```text
WorkerRun is the execution lease unit.
TaskAttempt is the product attempt/history unit.
Worker claim/heartbeat updates WorkerRun liveness, not only TaskAttempt lease.
```

### P1-2: WorkerRun heartbeat and lease fields

Later add:

```text
WorkerRun.last_heartbeat_at
WorkerRun.lease_expires_at
WorkerRun.heartbeat_source
```

TaskAttempt `lease_expires_at` can remain as an aggregate field or be migrated later once WorkerRun lease is authoritative.

### P1-3: durable System ToolCall boundary

Auto-drain and future recovery should use durable System ToolCalls with idempotency where practical.

### P1-4: process runtime lifecycle

ProcessRun has status and timeout, but needs a clearer long-term lifecycle for observe/attach, incremental logs, and terminal propagation to WorkerRun and TaskAttempt.

### P1-5: contract and UI state hardening

Shared contracts should eventually use explicit status unions for TaskAttempt, WorkerRun, GitWorktree, ApprovalRequest, and ProcessRun. UI should not infer important recovery states from loose strings.

### P2-1: artifact retention and disk pressure policy

ArtifactRef has `retention_policy`, but runtime retention is not enforced. Failed or interrupted evidence must not be removed silently.

### P2-2: Owner runtime durability

Owner AgentRun should later gain durable tool-loop checkpointing, interruption, resume, and event streaming. LangGraph-style checkpoint/interrupt concepts are useful references, but internal AgentRun/ToolCall should remain source of truth.

## Recommended implementation sequence

1. PR #16A: queued-state transition consistency.
2. PR #16B: stale-running recovery primitive.
3. PR #17: WorkerRun failure semantic normalization.
4. PR #18: WorkerRun heartbeat/lease fields.
5. PR #19: scheduler/System ToolCall recovery loop.
6. PR #20: contract/UI state hardening.

## Bottom line

The platform is moving in the right direction: durable TaskAttempt, WorkerRun, ProcessRun, ToolCall, AuditEvent, Worktree, Approval, and Artifact records are present.

The biggest long-term risk is having multiple overlapping execution semantics. The next design and implementation work should consolidate around one reliability spine:

```text
ToolCall -> TaskAttempt -> WorkerRun -> ProcessRun/APIRun -> Artifact/Audit/Event
```

with explicit transitions, WorkerRun lease/liveness, evidence preservation, and no automatic retry until idempotency is declared.
