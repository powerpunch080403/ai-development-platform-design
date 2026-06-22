---
type: adr
status: accepted
date: 2026-06-23
scope:
  - personal-mode
  - completion
  - roadmap
  - architecture
---

# ADR-0015 Personal Mode Completion Scope and Design Roadmap

## Status
Accepted

## Context
- 현재까지 Personal Mode MVP의 핵심 아키텍처와 도메인 모델, 그리고 AGY Worker Alpha에 대한 정책이 ADR-0005부터 ADR-0014를 통해 정의되었다.
- "설계 먼저, 구현 나중" 원칙에 따라, 다음 구현 단계로 넘어가기 전에 개인 모드가 "완성"되었다고 할 수 있는 기준을 설계로 정의해야 한다.
- 이번 완성 정의는 팀 모드나 중앙 권한 제어(Central Authority)를 제외한 오직 **Personal Mode Complete** 범위로 한정한다.

## Decision
개인 모드의 완성(Completion) 기준을 구체화하고, 이를 달성하기 위해 필요한 기능 영역과 설계/구현 로드맵을 정의한다.

## Personal Mode Complete Definition

Personal Mode is complete when a single local user can safely use the platform as their primary AI development workspace for real local projects, through Owner-led planning, controlled Worker execution, review, approval, merge, recovery, and local runtime management, without relying on Team Mode or Central Authority.

**개인 모드 완성은 단순히 Worker가 한 번 코드를 고치는 것이 아니다.**
사용자가 로컬 환경에서 실제 프로젝트를 등록하고, Owner와 대화하고, Task가 생성되고, Worker가 격리된 worktree에서 실행되고, 실패를 복구하고, 결과를 검토하고, 승인 후 병합하고, 기록과 산출물을 추적할 수 있는 상태를 말한다.

## Completion Areas

개인 모드 완성은 다음 15가지 영역으로 구성된다.

### A. Local Runtime and App Shell
- App-managed local runtime
- Browser-accessible Web UI first
- Desktop App Shell later
- app data directory
- local SQLite database
- runtime health/status
- local config
- safe startup/shutdown

### B. Identity, Device, Session
- local user bootstrap
- pairing code
- session token
- device/session list
- revoke session/device
- local-only auth

### C. Project and Repository Management
- Project registration
- multi-repository project support
- primary/supporting/docs/infra repository roles
- Git status
- dirty check
- default branch/base commit tracking
- safe repository refresh

### D. Conversation and Owner Experience
- Conversation belongs to project or general context
- User talks to Owner
- Owner keeps conversation history separate from run state
- Owner creates Agent Runs
- Owner explains plan, asks clarification when needed, and requests approval when policy requires it

### E. Work Item / Task / Task Attempt
- Work Item hierarchy
- Task as Worker assignment unit
- Task Attempt as one execution record
- retry without overwriting history
- Task instruction
- read_scope/write_scope/forbidden_scope
- risk/capability metadata

### F. Owner Tool Contract and Policy
- Owner cannot directly mutate DB/Git/filesystem
- Tool Call Envelope
- Tool Registry / Action Catalog
- idempotency key
- audit events
- Local Policy Engine
- Approval Interruption
- Owner Grant
- risk levels

### G. Worker and Adapter System
- Worker registry
- Worker claim/lease/heartbeat
- Manual Worker Adapter
- Mock Worker Adapter
- External CLI Adapter contract
- AGY Worker Alpha
- future Codex CLI adapter
- active WorkerRun guard

### H. Git Worktree and Branch Lifecycle
- app-managed isolated worktree
- task/attempt branch naming
- no source branch mutation by Worker
- result commit on attempt branch
- result diff
- source repo dirty block before Worker execution
- stale base detection

### I. Review, Approval, Merge
- Owner review
- diff inspection
- artifact inspection
- approval request
- approval fingerprint
- squash merge
- no Worker direct merge
- merge blocked on dirty/stale/unapproved state

### J. Failure, Recovery, Resume, Cleanup
- failed worktree preservation by default
- timeout/failure/scope violation worktree remains inspectable
- manual continuation
- retry attempt
- abandon
- cleanup only after explicit request or successful merge cleanup_pending
- Codex-like "partial work remains" behavior

### K. Artifact and Audit Trail
- artifact_ref
- diff patch
- stdout/stderr logs
- test/build/lint logs
- worker report
- error log
- process run transcript
- audit event
- policy decision record

### L. Process Runner and Local Command Safety
- executable + argument array
- shell=True forbidden
- stdin DEVNULL for background processes
- timeout/cancellation
- environment allowlist
- secret redaction
- process run record

### M. Testing and Validation
- Golden Path README update
- small function edit
- small app/game creation
- success/review/merge E2E
- write_scope violation E2E
- timeout/process failure E2E
- default tests skip real external CLI
- opt-in real CLI tests

### N. Settings and User Control
- local config
- autonomy profile
- approval preferences
- danger flag local config only
- cleanup preferences
- adapter enable/disable
- default write_scope policy

### O. Documentation and Operability
- first-run guide
- project registration guide
- Worker/Owner mental model
- recovery guide
- approval/risk guide
- troubleshooting guide
- backup/export notes

## Required Design Work Before Implementation
구현을 진행하기 전, 다음 항목들에 대한 상세 설계(ADR 후보)가 확정되어야 한다. (본 ADR에서는 후보로만 지정하며, 실제 작성은 이후 수행한다.)

- ADR-0016 Personal Mode Complete UX and Owner Conversation Flow
- ADR-0017 Personal Mode Failure Recovery and Worktree Resume Policy
- ADR-0018 Personal Mode Settings, Grants, and Autonomy Profiles
- ADR-0019 Personal Mode Artifact, Audit, and Observability Model
- ADR-0020 Personal Mode Desktop Shell and Local Runtime Packaging

## Non-goals
- Team Mode 설계 확정
- Central Authority 서버 구축 및 통합
- Remote Runner 세부 설계 확정
- Desktop Shell의 구체적 기술 스택(Tauri, Electron 등) 확정
- Autonomy profile 세부 파라미터 확정

## Consequences
- 개인 모드 완성이라는 뚜렷한 목표가 세워짐에 따라, 불필요한 팀 기능 개발을 미루고 로컬 개발 루프의 안정화에 집중할 수 있다.
- 실패 복구, 정책 엔진, Worktree 라이프사이클 등 복잡한 내부 구조를 먼저 견고하게 설계해야 하므로 초기 아키텍처 개발 비용이 크다.

## Related ADRs
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]
- [[07 ADR/ADR-0013 MVP Implementation Slice and Repository Strategy]]
- [[07 ADR/ADR-0014 Owner-Led AGY Worker Alpha and Safe Pilot Policy]]
- [[07 ADR/ADR-0016 Initial Personal Mode MVP Execution Policy]]
- [[07 ADR/ADR-0017 Failure Recovery and Worktree Resume Policy]]
