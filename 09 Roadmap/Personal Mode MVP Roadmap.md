# Personal Mode MVP Roadmap

**Status**: Implementation can resume after ADR-0017.
*Note: ADR-0016/0017 were updated from product-owner review.*

## Recommended implementation sequence after design readiness:
1. Owner-Led MVP Execution Alignment
2. MVP UI State Surface
3. Real AI Worker MVP Checkpoint
4. Dogfood Game Project

The dogfood checkpoint validates that Personal Mode is useful enough for the product owner to use the platform to build a real small game project.

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]
- [[07 ADR/ADR-0011 Personal Runtime, Account, Device, and Session Model]]
- [[07 ADR/ADR-0012 Remote Test Runner Worker Capability]]
- [[07 ADR/ADR-0013 MVP Implementation Slice and Repository Strategy]]
- [[07 ADR/ADR-0014 Owner-Led AGY Worker Alpha and Safe Pilot Policy]]
- [[07 ADR/ADR-0015 Personal Mode Completion Scope and Design Roadmap]]
- [[07 ADR/ADR-0016 Initial Personal Mode MVP Execution Policy]]
- [[07 ADR/ADR-0017 Failure Recovery and Worktree Resume Policy]]
- [[07 ADR/ADR-0018 Personal Mode Approval Modes, Grants, and Autonomy Profiles]]

## Phase A: Local Runtime and Identity Complete
(Already validated / Implementation baseline)

- Public v2 Monorepo and Skeleton
- `apps/server`, `apps/web` 기본 구조
- SQLite + SQLAlchemy 2 + Alembic baseline
- 단일 로컬 사용자 자동 생성 및 local user 연결 모델
- 연결된 장치와 세션 (pairing code, opaque Session token)
- 허용 프로젝트 루트 및 운영체제별 경로 검증
- Browser-accessible Web UI first

## Phase B: Project/Repository and Conversation Complete
(Already validated / Implementation baseline)

- Project와 ProjectRepository 모델
- 기존 Git 저장소 가져오기
- Conversation과 Message 관리
- User talks to Owner
- Owner keeps conversation history separate from run state
- Conversation belongs to project or general context

## Phase C: Owner Tool/Policy Complete
(Already validated / Implementation baseline)

- Owner Supervisor 및 Agent Run/Step
- Owner Tool Contract (Owner cannot directly mutate DB/Git/filesystem)
- Tool Registry / Action Catalog
- Tool Call Envelope 및 idempotency key
- Local Policy Engine 및 Approval Interruption
- Owner Grant 및 Risk levels 기본 구조
- Owner Runtime Provider abstraction
- Codex CLI Owner provider MVP
- Future API Owner provider

## Phase D: Work Item/Task/Attempt Complete
(Implementation in progress)

- Work Item hierarchy 설계
- Task as Worker assignment unit
- Task Attempt as one execution record
- Task instruction, read_scope, write_scope, forbidden_scope (Closed by ADR-0016 Initial MVP Execution Policy)
- risk/capability metadata 연결
- retry without overwriting history
- Owner planning and Task decomposition (Closed by ADR-0016)

## Phase E: Worker/Adapter Complete
(Implementation in progress)

- Worker registry
- Worker claim/lease/heartbeat
- Manual/Mock Worker Adapter (Baseline)
- External CLI Adapter contract
- AGY Worker Alpha Integration (Opt-in, controlled)
- AGY Alpha Process stdin DEVNULL stabilization
- active WorkerRun guard
- Owner preflight policy (Closed by ADR-0016)

## Phase F: Git Worktree/Review/Merge Complete
(Implementation in progress)

- app-managed isolated worktree
- task/attempt branch naming
- no source branch mutation by Worker
- result commit on attempt branch
- result diff
- source repo dirty block before Worker execution
- stale base detection
- Owner review, diff/artifact inspection
- approval request, squash merge, merge block conditions

## Phase G: Failure Recovery and Resume Complete
(Next up)

- failed worktree preservation by default (Closed by ADR-0017)
- timeout/failure/scope violation worktree remains inspectable (Closed by ADR-0017)
- manual continuation (Closed by ADR-0017)
- retry attempt (Closed by ADR-0017)
- abandon (Closed by ADR-0017)
- cleanup only after explicit request or successful merge cleanup_pending (Closed by ADR-0017)
- Codex-like "partial work remains" behavior (Closed by ADR-0017)

## Phase H: Artifact/Audit/Observability Complete
(Next up)

- `artifact_ref` 도입
- diff patch, stdout/stderr logs, test/build/lint logs 보관
- worker report, error log, process run transcript
- audit event 기록
- policy decision record 시스템
- Remote Test Runner Node 등록 및 artifact upload 통합 (Phase 4B)

## Phase I: Settings/Autonomy/Profile Complete
(Next up)

- local config
- autonomy profile (Ask for approval, Approve on my behalf 등) (Closed by ADR-0018)
- approval preferences (Closed by ADR-0018)
- danger flag local config only
- cleanup preferences
- adapter enable/disable
- default write_scope policy (Closed by ADR-0016)
- Initial MVP UI minimums (Closed by ADR-0016)

## Phase J: Desktop Shell and Packaging Complete
(Next up)

- Desktop App Shell (Tauri/Electron 등)에서 재사용 가능한 UI 구성
- Desktop packaging 및 OS keychain 연동 (향후 확장 목표)
- 안전한 시작/종료 로직 (safe startup/shutdown)

## Phase K: Personal Mode Complete Acceptance Tests
(Validation)

- 실제 저장소 테스트 (Ubuntu reference / Windows 실행 검증)
- 재시작 복구, CLI 로그인 만료 테스트
- Golden Path README update E2E
- small function edit E2E
- small app/game creation E2E
- success/review/merge E2E
- write_scope violation E2E
- timeout/process failure E2E
- default tests skip real external CLI / opt-in real CLI tests
- Safe Pilot Policy validation for AGY Alpha

## Deferred

- 팀 모드와 중앙 Authority
- PostgreSQL 연동
- 여러 Worker Host 및 일반 Worker Task의 분산
- 전문 Worker 분리
- local IPC 사용 및 자동 업데이트
- 중앙 account 로그인과 다중 기기 동기화
- Background Service 설치 방식
- Remote AI Worker Node
- macOS 공식 지원
