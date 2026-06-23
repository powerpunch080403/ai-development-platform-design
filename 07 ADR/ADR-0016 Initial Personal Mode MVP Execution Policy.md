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

### Project Working Scope
- Project working scope is an optional project-level boundary for Owner activity.
- It may be set during project registration or later in project settings.
- It can include existing files, folders, or planned files/folders.
- It may be left unset.
- Unset project working scope means there is no project-level restriction yet; it does not mean Workers can mutate everything without Task scope.
- Owner uses project working scope when planning Work Items and Tasks.
- If Owner needs to act outside project working scope, Owner must request approval unless the active approval mode/grant allows it.

### Task Write Scope
- Worker-executable Task should have an Owner-assigned Task write_scope.
- Task write_scope is the concrete boundary passed to Worker.
- Worker must not expand it.
- Task write_scope should normally be inside project working scope when one exists.
- Broad Task write_scope is allowed only when Owner explains why and approval mode/policy allows it.
- User does not manually select files for every Task by default.
- read_scope may be broader than write_scope.
- forbidden_scope must always include secrets and runtime/private files.

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

- The default posture is conservative: Worker should not create new files unless Owner's Task explicitly allows it.
- Owner may allow new files when the Task requires new project files.
- Protected path creation or modification requires explicit Owner intent and policy/grant approval.
- Approval mode may relax repeated prompts but does not let Worker decide protected access by itself.
- If AGY/Worker creates files outside the Task write_scope, the result commit is rejected.

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

## Approval Modes
Initial MVP execution policy depends on the active approval mode:
- Ask for approval
- Approve on my behalf
- Full access
- Custom

Detailed grants and autonomy profiles are deferred to ADR-0018.

## Initial MVP Execution Defaults
- Manual and Mock Worker remain baseline validation adapters.
- AGY or another real AI Worker path is an important initial MVP checkpoint for proving Owner/Worker operation.
- AGY remains opt-in and controlled until future ADRs open broader automation.
- External CLI dry-run remains diagnostic/contract validation capability.
- Real external CLI tests are skipped by default.
- Worker cannot run without TaskAttempt.
- Worker cannot merge.
- Worker cannot push.
- Worker cannot mutate source/default branch directly.
- Worker result commit is created only after write_scope validation.
- Owner review is required before merge.
- Personal Mode MVP default merge is squash merge with approval policy.

## Initial MVP UI Minimums
Initial MVP UI should expose:
- Project / Repository
- Conversation / Owner
- Work Item / Task
- Task Attempt
- WorkerRun
- Worktree
- Diff
- Artifact
- Logs
- Approval
- Settings summary

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
- [[07 ADR/ADR-0017 Failure Recovery and Worktree Resume Policy]]
- [[07 ADR/ADR-0018 Personal Mode Approval Modes, Grants, and Autonomy Profiles]]
