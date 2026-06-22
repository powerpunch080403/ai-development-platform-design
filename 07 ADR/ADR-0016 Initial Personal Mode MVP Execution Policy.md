---
type: adr
status: accepted
date: 2026-06-23
scope:
  - personal-mode
  - mvp
  - execution-policy
  - task-decomposition
  - write_scope
  - preflight
---

# ADR-0016 Initial Personal Mode MVP Execution Policy

## Status
Accepted

## Context
- ADR-0014 clarified the Owner-led Worker model. User talks to Owner; Worker receives Owner-created Tasks only.
- Initial Personal Mode MVP needs enough execution policy to safely resume implementation.
- Recent design gap audit (`11 Reviews/Initial Personal Mode MVP Design Gap Audit.md`) identified undecided execution blockers such as Owner planning scope, Task decomposition, default write_scope, and preflight details.
- This ADR closes only the execution-policy blockers required for initial MVP implementation.
- Failure recovery/resume semantics are deferred to ADR-0017.

## Decision
We define the initial execution policy for Owner planning, Task decomposition, write_scope defaults, protected paths, Owner preflight, and minimum UI requirements to unblock MVP implementation. 

## Owner Planning and Task Creation
- User may send broad natural-language requests to Owner.
- Owner must not pass broad user requests directly to Worker.
- Owner converts user requests into Work Items and Tasks.
- A Worker can only run after a Task exists.
- Every Worker-executable Task must include:
  - task instruction
  - target project
  - target repository
  - read_scope
  - write_scope
  - forbidden_scope
  - risk level
  - required capability / adapter kind
  - approval requirement

Owner planning 최소 규칙:
1. **Simple explicit request**
   - Example: "README.md에 설치 방법 한 줄 추가해줘."
   - Owner may create a single Task directly.
   - write_scope must be narrow and explicit.

2. **Broad but low-risk request**
   - Example: "README 문서 좀 정리해줘."
   - Owner should create or update a Work Item and then create one or more Tasks.
   - Owner may ask clarification if target files are unclear.

3. **Ambiguous, multi-file, or bug-fix request**
   - Example: "로그인 버그 고쳐줘."
   - Owner must perform planning/analysis before Worker code modification.
   - Planning may be an Owner reasoning step, an analysis Task, or a read-only Worker Task.
   - Write Task is created only after likely target files/scopes are known.

4. **High-risk request**
   - Example: dependency/config/migration/security/auth/system changes.
   - Owner must classify risk and request approval when policy requires.

- Owner may create Tasks automatically only within current policy/grant/risk limits.
- Broad user approval does not give Workers unlimited authority.
- Worker cannot create new Tasks for itself.

## Initial Task Scope Policy
- Worker-executable Task must have explicit write_scope.
- No implicit write_scope default is allowed for Worker execution.
- "." or whole-repository write_scope is not allowed by default in initial MVP.
- Whole-repository write_scope requires explicit Owner explanation and user approval.
- write_scope should be as narrow as practical.
- read_scope may be broader than write_scope.
- forbidden_scope must always include secrets and runtime/private files.

Allowed narrow examples:
- README.md
- docs/
- apps/server/src/auth.py
- apps/server/tests/test_auth.py

Broad examples requiring approval:
- .
- src/
- apps/
- package manager files plus source files
- migrations plus model files

## Protected Paths and New Files
Protected by default:
- .env
- .env.*
- secrets
- credentials
- private keys
- local SQLite DB files
- runtime-data
- generated artifacts
- worktrees
- logs with sensitive data
- OS/system directories

High-risk categories requiring elevated risk / approval:
- authentication/authorization code
- security policy code
- dependency manifest or lock files
- database migrations
- CI/CD configuration
- deployment configuration
- shell scripts / executable scripts
- package manager configuration
- files that affect broad runtime behavior

- Initial MVP default: allow_new_files = false.
- Owner may set allow_new_files = true only when the Task explicitly requires it.
- New files in protected categories remain forbidden even if allow_new_files = true.
- If AGY/Worker creates files outside write_scope, result commit is rejected.

## Owner Preflight Policy
Preflight is Owner's execution readiness check before Worker execution.
It is not a mandatory user confirmation button for every Worker run.

Before Worker execution, Owner/Local Control Plane must check:
- Project exists
- Repository exists
- Repository working tree is clean
- Target branch and base commit are known
- Task exists
- Task has explicit instruction
- Task has explicit write_scope
- forbidden_scope is present
- risk level is assigned
- required adapter/capability is available
- active WorkerRun does not already exist for the Attempt
- Worker lease/claim can be acquired
- worktree can be created or reused according to policy
- approval requirement is satisfied or Approval Interruption is raised
- danger flag usage is permitted by local config / grant / policy

- Initial MVP does not require a separate PreflightRecord table.
- Preflight results may be recorded through PolicyDecision, AuditEvent, ToolCall, AgentRunStep, WorkerRun, or ProcessRun records.
- A later ADR may introduce a dedicated PreflightRecord if needed.

- If preflight fails before Worker starts, WorkerRun must not start.
- TaskAttempt should transition to blocked or failed depending on whether user action can resolve it.
- Task should transition to blocked if user action or project state cleanup is required.
- AgentRun should enter waiting_for_user when user action/approval is needed.
- AgentRun should fail only for non-recoverable system errors.

## Initial MVP Execution Defaults
- Manual Worker and Mock Worker remain valid MVP baseline adapters.
- AGY Worker remains opt-in alpha capability, not required for MVP core completion.
- External CLI dry-run remains diagnostic/contract validation capability.
- Real external CLI tests are skipped by default.
- Worker cannot run without TaskAttempt.
- Worker cannot merge.
- Worker cannot push.
- Worker cannot mutate source/default branch directly.
- Worker result commit is created only after write_scope validation.
- Owner review is required before merge.
- Personal Mode MVP default merge is squash merge with approval policy.

AGY Alpha may be included in MVP as controlled opt-in capability, but general free-form AGY automation is not part of initial MVP acceptance.

## Initial MVP UI Minimums
Initial MVP should expose at least:
1. Project / Repository registration and status
2. Conversation / Owner interaction view
3. Work Item / Task list and detail
4. Task Attempt / WorkerRun / Worktree status
5. Diff and Artifact review view
6. Approval request card
7. Merge result / cleanup_pending status
8. Basic settings/status view:
   - local session/device
   - adapter availability
   - autonomy/profile summary
   - danger flag read-only status

- failed/timed_out/scope-violating Attempt status
- worktree path or managed reference
- last diff/status if available
- error artifact/log reference

## Consequences
- The Owner-led logic requires parsing user requests to safely scope down tasks, leading to complex prompts for the Owner LLM, but increases execution safety and avoids unlimited scope destruction.
- We establish the baseline requirements for MVP UI implementation, ensuring critical states are surfaced without needing complete Desktop parity immediately.
- A strict separation of Owner execution intent (Tasks) and Worker constraints prevents out-of-scope code changes automatically.

## Non-goals
- 실패 worktree resume 의미론 확정 (ADR-0017에서 작성 예정)
- autonomy profile 세부값 및 승인 fingerprint 상세 형태 확정
- Desktop Shell 기술 선택
- Remote Runner 세부 설계 확정
- Team Mode 설계 변경

## Related ADRs
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]
- [[07 ADR/ADR-0014 Owner-Led AGY Worker Alpha and Safe Pilot Policy]]
- [[07 ADR/ADR-0015 Personal Mode Completion Scope and Design Roadmap]]
