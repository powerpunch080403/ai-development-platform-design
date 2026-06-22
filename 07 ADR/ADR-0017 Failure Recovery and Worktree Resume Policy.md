---
type: adr
status: accepted
date: 2026-06-23
scope:
  - personal-mode
  - mvp
  - failure-recovery
  - worktree-resume
  - task-attempt
---

# ADR-0017 Failure Recovery and Worktree Resume Policy

## Status
Accepted

## Context
- ADR-0009 defines Task Attempt as one execution record and retry must not overwrite history.
- ADR-0014 states failed worktrees are preserved by default.
- ADR-0016 defers failure recovery/resume semantics to ADR-0017.
- Codex-like behavior is desired: partial files remain inspectable after error, timeout, or quota stop.
- Initial MVP needs deterministic rules for retry, manual continuation, abandon, and cleanup.

## Decision
We define the classes of failures, the strict preservation policy of failed worktrees, the semantics for retry and manual continuation, the difference between abandon and cleanup, the specific state transitions, and the UI/Owner responsibilities for the Initial MVP.

## Failure Classes
Initial MVP categorizes failures into the following classes:

1. **preflight_blocked**
   - Worker never started.
   - Examples: dirty source repo, missing write_scope, missing approval, active WorkerRun exists.
   - Policy: no worktree changes expected; user/Owner resolves condition and retries.

2. **worker_failed**
   - Worker started but failed.
   - Examples: CLI non-zero exit, adapter error, model/tool error.
   - Policy: worktree preserved by default.

3. **timed_out**
   - Worker or ProcessRun exceeded timeout.
   - Policy: worktree preserved by default.

4. **scope_violation**
   - Worker modified or created files outside write_scope or forbidden_scope.
   - Result commit is rejected.
   - Policy: worktree preserved by default.

5. **interrupted**
   - User/Owner cancelled, process interrupted, app shutdown, quota stop, or external limit.
   - Policy: worktree preserved by default.

6. **review_rejected**
   - Worker produced a result commit, but Owner rejected it during review.
   - Policy: result commit/artifacts preserved; follow-up Task or retry can be created.

7. **merge_blocked**
   - Result is accepted but cannot merge due to dirty source, stale base, missing approval, or policy issue.
   - Policy: result commit preserved; Owner resolves merge blocker or requests user action.

## Preserved Worktree Policy
- Failed worktrees are preserved by default.
- Preserved means the app-managed worktree directory, branch, diff/status, logs, and artifacts remain available for inspection.
- Preservation does not mean the failed Attempt can be freely mutated forever.
- The preserved worktree is evidence and recovery context.
- Automatic cleanup must not remove failed/timed_out/interrupted/scope-violating worktrees.
- Cleanup is explicit user/Owner action or successful merge cleanup flow.
- The platform may later offer retention policies, but MVP default is preserve.

## Retry and Resume Semantics
- A Task Attempt is one execution record.
- A failed Attempt is not overwritten.
- Automatic Worker retry creates a new Task Attempt.
- Owner may use preserved artifacts/diff/logs from the failed Attempt to create the next Task Attempt.
- The new Attempt should reference the previous Attempt as `retry_of_attempt_id` or `parent_attempt_id` if supported.
- If the schema does not yet support `retry_of_attempt_id`, the relationship can be recorded in audit/artifact/AgentRunStep until later migration.
- Same-Attempt re-execution is not the default MVP behavior.
- Same-Attempt continuation may be added later only if state/audit semantics are explicitly defined.
- MVP should avoid mutating failed Attempt history.
- `retry_requested` means Owner has decided a retry should be planned.
- It does not mean the same Attempt will run again.
- It should normally lead to creating a new Task Attempt.

## Manual Continuation
- Manual continuation means the user/Owner can inspect preserved files and decide how to proceed.
- Initial MVP should not silently turn a failed Attempt into a successful Attempt by editing the same history record.
- Manual continuation should create a new explicit continuation Attempt or a follow-up Task.
- The continuation Attempt may start from:
  1. original base commit,
  2. failed worktree branch,
  3. failed result branch if result commit exists,
  depending on risk and policy.
- For safety, continuation should start as a new Attempt.
- Owner must explain whether it is using the failed worktree as reference only or as starting point.
- If using failed worktree as starting point, Owner should request approval when the prior state includes unreviewed broad changes.

## Abandon and Cleanup
Abandon:
- Logical decision to stop pursuing an Attempt/worktree.
- Keeps audit history and may keep artifacts.
- May mark worktree as abandoned or cleanup_pending depending on policy.

Cleanup:
- Physical removal of app-managed worktree directory.
- Must preserve database records, audit events, artifacts, commit SHAs, and review history.
- Must be path-safe and never delete source repo.

Policy:
- abandon does not necessarily delete files immediately.
- cleanup requires explicit Owner/user action or successful merge cleanup flow.
- cleanup can be offered for abandoned/merged/failed worktrees, but failed cleanup should require user/Owner intent.
- cleanup must never run automatically just because a Worker failed.

## State Transitions
Task Attempt:
- `worker_failed`: Worker started and failed.
- `failed`: terminal system failure state.
- `retry_requested`: Owner plans a new Attempt.
- `rejected`: Owner rejected result after review.
- `abandoned`: Owner/user stopped pursuing this Attempt.
- `cancelled`: user/Owner cancellation before completion.
- `merge_ready`: accepted and ready for merge.
- `merged`: result merged to source branch.

Worktree:
- `dirty_result`: Worker changed files but result commit not yet accepted.
- `committed`: result commit exists on attempt branch.
- `reviewing`: Owner is reviewing result.
- `cleanup_pending`: successful merge or explicit abandon/cleanup decision allows cleanup.
- `abandoned`: no longer being pursued, but may still exist until cleanup.
- `cleaned`: app-managed worktree directory removed safely.
- `failed`: worktree operation failed; path may or may not exist.

Task:
- `blocked`: user/project state/action is required before progress.
- `changes_requested`: Owner rejected result and follow-up work is needed.
- `failed`: Task cannot continue without new decision.
- `completed`: accepted result merged or task resolved according to policy.

- Failure status does not imply physical cleanup.
- cleaned is a physical cleanup result, not a normal failure state.

## UI and Owner Responsibilities
Owner should explain:
- what failed,
- whether files were preserved,
- what can be inspected,
- recommended next action,
- whether retry will create a new Attempt,
- whether cleanup/abandon is safe.

UI should show at least:
- failure class,
- Attempt status,
- Worktree status,
- worktree path or managed reference,
- diff/status if available,
- process/worker error artifact,
- retry/abandon/cleanup availability as actions or Owner commands.

Actions may be exposed through Owner conversation first.
Dedicated buttons may be added later.

## Consequences
- Preserving failed worktrees by default increases disk space usage but ensures safe Codex-like behavior and strong auditability.
- Requiring a new Attempt for retry or resume preserves the immutable history of execution, simplifying the data model.
- The clear distinction between abandon (logical state) and cleanup (physical deletion) avoids unexpected loss of forensic data.

## Non-goals
- retention/pruning 세부 정책 확정
- Desktop Shell 기술 선택
- Team Mode 변경
- Remote Runner 세부 설계 변경

## Related ADRs
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0014 Owner-Led AGY Worker Alpha and Safe Pilot Policy]]
- [[07 ADR/ADR-0016 Initial Personal Mode MVP Execution Policy]]
