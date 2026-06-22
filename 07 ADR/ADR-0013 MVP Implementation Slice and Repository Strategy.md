---
type: adr
status: accepted
date: 2026-06-22
scope:
  - personal-mode
  - mvp
  - implementation
  - repository
  - monorepo
  - backend
  - frontend
  - worker
  - adapter
  - golden-path
---

# ADR-0013: MVP Implementation Slice와 Repository 전략

## 배경

[[07 ADR/ADR-0001 Central Authority]]부터 [[07 ADR/ADR-0004 Governance and Approval Policy]]까지는 중앙 권위, Personal Owner, Discord 제거와 거버넌스를 정했다. [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]부터 [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]까지는 Personal Runtime, Agent Run과 승인 경계를 정했다. [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]부터 [[07 ADR/ADR-0012 Remote Test Runner Worker Capability]]까지는 Personal Mode MVP, 데이터 모델, Tool Contract, Local Runtime, Device/Session과 Remote Test Runner를 구체화했다.

이제 설계 저장소와 v1 참고 구현을 유지하면서 새 v2 구현 저장소를 어떤 구조로 만들고, 첫 구현 Slice를 어디까지 자를지 결정해야 한다.

## 문제

설계 전체를 한 번에 구현하거나 v1 저장소를 점진적으로 바꾸면 다음 문제가 생긴다.

- v1의 Discord, 동기식 Owner CLI, 기존 DB와 역할 구조가 새 설계에 스며들 수 있다.
- 외부 AI CLI 통합에 막혀 핵심 상태 머신과 Git 흐름 개발이 중단될 수 있다.
- Remote Test Runner와 Desktop Shell을 먼저 만들면 기본 Local Worker 경로가 늦어진다.
- server, web, contracts와 adapters가 초기에 여러 저장소로 흩어져 변경 속도가 느려질 수 있다.
- Public 저장소에 runtime DB, artifact, worktree나 credential이 유입될 수 있다.
- “MVP 전체”와 “첫 구현 Slice”가 혼합되어 완료 기준이 불명확해질 수 있다.

## 결정

- v1 저장소를 수정하거나 fork하지 않고 새 Public v2 구현 저장소 `ai-development-platform`을 만든다.
- 설계 원본은 `ai-development-platform-design`, v1은 `ai-game-company-server`, 구현은 `ai-development-platform`으로 분리한다.
- v2는 apps/packages 경계가 있는 Monorepo로 시작한다.
- Backend는 Python + FastAPI, Frontend는 React + TypeScript + Vite를 사용한다.
- Python tooling은 uv, Frontend package manager는 pnpm을 사용한다.
- SQLite + SQLAlchemy 2 + Alembic으로 시작한다.
- REST JSON을 우선하고 SSE를 상태·로그 streaming 후보로 둔다. WebSocket은 후속이다.
- Ubuntu Desktop을 primary runtime target, Windows를 development/compatibility target으로 둔다.
- 목표 Adapter는 Codex CLI Owner와 Antigravity CLI Worker다. Mock과 Manual Adapter를 pipeline 개발과 복구용으로 함께 둔다.
- 첫 구현 Slice는 Local Worker Golden Path 1을 완성하는 데 집중한다.
- Codex/Antigravity 통합은 기본 pipeline 뒤에, Remote Test Runner는 그 다음에 구현한다.
- 초기 UI는 browser-accessible Desktop-ready Web UI이며 Desktop App Shell은 후속이다.

### 기존 ADR과의 관계

- ADR-0008의 Windows와 Linux 지원 목표를 유지한다. Ubuntu Desktop primary는 첫 안정화와 검증 우선순위이며 Windows 지원 철회가 아니다.
- ADR-0009의 Task, Attempt, Worktree, 자동 commit과 squash merge 상태 흐름을 첫 Slice로 구현한다.
- ADR-0010의 Tool Contract, Worker claim/lease와 artifact 경계를 사용한다.
- ADR-0011의 Web UI pairing/Session을 첫 Slice에 포함하고 Desktop App Shell은 후속으로 유지한다.
- ADR-0012의 Remote Test Runner는 MVP 전체에 포함하되 Local Worker Golden Path 이후 구현한다.

## v2 구현 저장소 전략

```text
v1 저장소를 수정해서 v2를 만들지 않는다.
새 v2 구현 저장소를 만든다.
```

저장소 관계:

```text
ai-development-platform-design
= 설계 / ADR / architecture / roadmap 저장소

ai-development-platform
= 새 v2 구현 저장소

ai-game-company-server
= v1 참고 구현 저장소
```

v2는 v1을 fork하지 않고 v1을 직접 수정하지 않으며 코드를 그대로 복사하지 않는다. v1은 누락 요구사항 발견, 아이디어와 재작성 후보 분석에만 사용한다. 충돌하면 ADR-0001~0013과 현재 도메인 모델을 우선한다.

이 ADR은 구현 저장소의 생성 방식을 결정하지만 실제 `ai-development-platform` 저장소를 생성하지 않는다.

## v1 저장소와의 관계

v1의 FastAPI, SQLite, Worker lease, Test Runner, Artifact와 Approval 아이디어는 참고 후보가 될 수 있다. 참고 여부가 새 제품 기능 채택을 뜻하지 않는다.

다음은 그대로 가져오지 않는다.

- Discord 결합 구조
- 동기식 Owner CLI 실행 구조
- 기존 DB schema와 역할·권한 모델
- 기존 서버 topology
- 기존 Golden Path 시나리오

Golden Path는 v2의 실제 상태 머신과 사용자 흐름을 기준으로 다시 정의한다.

## 저장소 공개 범위

새 `ai-development-platform` 저장소는 Public으로 만든다. 공개 저장소는 설계 검토, 외부 링크 기반 review와 사용자·Codex 협업을 단순화한다.

Public source repository에 다음을 commit하지 않는다.

```text
secrets
API keys
.env와 환경별 secret 파일
local SQLite DB
사용자 Project 내용
generated artifacts
Git worktrees
runtime logs
local credentials와 session tokens
```

Skeleton은 강한 `.gitignore`와 값이 비어 있는 `.env.example`을 포함한다. `.env.example`은 commit한다.

`.gitignore` 최소 범주:

```text
.env
.env.*
!.env.example
*.db
*.sqlite
*.sqlite3
data/
runtime-data/
artifacts/
worktrees/
logs/
.cache/
.venv/
node_modules/
dist/
build/
coverage/
```

## Monorepo 구조

```text
ai-development-platform/
  apps/
    server/
    web/
    runner-agent/
  packages/
    shared-contracts/
    cli-adapters/
  docs/
  scripts/
  tests/
  README.md
  AGENTS.md
```

책임:

| 경로 | 책임 |
| --- | --- |
| `apps/server` | App-managed Local Runtime, FastAPI, Local Control Plane, SQLite, Tool Contract와 Worker Supervisor |
| `apps/web` | Browser-accessible Desktop-ready Web UI |
| `apps/runner-agent` | Remote Test Runner Agent. Local Worker path 뒤에 구현 |
| `packages/shared-contracts` | API/Tool schema, DTO, shared types와 status enum |
| `packages/cli-adapters` | Codex, Antigravity, Mock와 Manual Adapter contract |
| `docs` | 실행 가이드와 developer/architecture note. 설계 원본은 design repo |
| `scripts` | Windows와 Linux 개발 script |
| `tests` | integration, Golden Path와 Adapter contract test |

`apps/server`와 `apps/web`은 shared contracts를 참조할 수 있다. CLI Adapter package는 외부 process 실행 계약을 담는다. apps/packages 경계를 유지해 필요하면 나중에 별도 repository나 package로 분리할 수 있다.

## 기술 스택

| 영역 | 결정 |
| --- | --- |
| Backend | Python + FastAPI |
| Frontend | React + TypeScript + Vite |
| Python tooling | uv |
| Frontend tooling | pnpm |
| Database | SQLite |
| ORM / migration | SQLAlchemy 2 + Alembic |
| API | REST JSON first |
| Streaming | SSE 후보, WebSocket 후속 |

Python과 FastAPI는 local file, Git, process runner, AI CLI와 SQLite 작업에 적합하다. React/TypeScript/Vite는 Desktop App Shell에서 재사용 가능한 Web UI에 적합하다. uv와 pnpm은 빠르고 재현 가능한 개발 환경을 제공한다. SQLAlchemy 2와 Alembic은 SQLite MVP와 향후 PostgreSQL 확장을 열어 둔다.

REST JSON은 구현과 debugging을 우선한다. SSE는 Agent Run 상태와 log streaming 후보로 두되 정확한 event contract는 후속이다. 양방향 streaming이 실제로 필요할 때 WebSocket을 검토한다.

## OS 타깃

```text
Primary runtime target: Ubuntu Desktop
Development and compatibility target: Windows
```

첫 MVP는 Ubuntu Desktop에서 안정 실행되는 것을 primary 검증 기준으로 한다. Windows에서도 개발, review와 기본 실행이 가능해야 한다. WSL은 공식 필수 전제가 아니다.

처음부터 OS Path Resolver와 Process Runner abstraction을 둔다. shell invocation, file locking, path separator, symlink/junction, worktree path, executable discovery와 process cancellation 차이를 격리한다. Ubuntu native와 Windows native 실행을 모두 고려한다.

## CLI Adapter 전략

```text
Target adapters are Codex CLI for Owner and Antigravity CLI for Worker.
Mock and Manual adapters exist only for development, pipeline testing, and recovery.
```

Owner와 Worker는 제품 도메인 역할이다. Codex와 Antigravity는 그 역할을 수행하는 초기 외부 Agent Process Adapter이며 Identity 자체가 아니다.

플랫폼은 외부 CLI 내부 state를 소유하지 않는다. executable, argument array, prompt/context, working directory, environment, timeout, stdout/stderr, exit status, generated artifact와 결과 Git diff/commit을 관찰하고 기록한다. Adapter는 다른 model/CLI로 교체할 수 있어야 한다.

초기 Adapter:

- **Codex CLI Owner Adapter**: Owner Agent Run의 목표 Adapter
- **Antigravity CLI Worker Adapter**: Worker Task Attempt의 목표 Adapter
- **Mock Adapter**: 실제 AI CLI 없이 deterministic 결과나 작은 변경을 만드는 test Adapter
- **Manual Adapter**: 플랫폼이 Attempt와 Worktree를 만들고 Human이 해당 Worktree에서 작업한 결과를 제출하는 임시 Adapter

외부 CLI의 auth, interactive prompt, structured output, cancellation과 failure detection은 실험이 필요하다. Mock과 Manual Adapter는 목표 방향을 대체하지 않으며 외부 CLI가 막혀도 Task → Attempt → Worktree → Commit → Review → Merge pipeline 개발이 계속되게 한다.

## 1차 MVP 구현 Slice

첫 구현 목표는 Local Worker Golden Path다. 처음부터 완전한 AI 자동화를 목표로 하지 않고 상태 머신과 Git/review/merge 흐름을 먼저 관통한다.

순서:

1. 새 v2 repository skeleton
2. uv/pnpm 개발 환경
3. FastAPI server skeleton
4. React/Vite web skeleton
5. SQLite + Alembic migration baseline
6. Local Runtime config와 app data directory
7. Web UI pairing code와 Session token
8. Project/Repository 등록
9. Git status와 dirty check
10. Conversation/Message 기록
11. Agent Run 기록
12. Tool Registry 최소 subset
13. Tool Call Envelope 저장
14. Work Item/Task/Task Attempt 생성
15. Worker registry와 claim/lease
16. Mock Worker Adapter
17. Manual Worker Adapter
18. Git worktree 생성
19. 작업 branch 생성
20. 작은 파일 수정 또는 Manual 변경 수집
21. Attempt 결과 자동 commit
22. `artifact_ref` 생성
23. Owner review 화면
24. diff 확인
25. squash merge 준비
26. 사용자 승인
27. squash merge 수행
28. runtime/audit event 기록

이 Slice가 완료된 뒤 Codex CLI Owner와 Antigravity CLI Worker를 붙이고, 다음으로 Remote Test Runner를 구현한다.

## Golden Path

### Golden Path 1: README 수정

```text
기존 Git repository 등록
→ “README에 한 줄 추가해줘” 요청
→ Owner가 Task 생성
→ Worker가 Task Attempt claim
→ worktree와 작업 branch 생성
→ Mock/Manual 또는 Worker Adapter가 README 수정
→ 작업 branch commit 생성
→ Owner review
→ 사용자 diff 확인과 squash merge 승인
→ main/default branch 반영
```

Golden Path 1은 첫 구현 Slice의 완료 기준이다.

### Golden Path 2: 작은 함수 수정

```text
작은 Python 또는 JavaScript repository 등록
→ 함수 수정 요청
→ Worker가 파일 수정
→ local test 또는 lint
→ artifact_ref 생성
→ Owner review
→ squash merge
```

### Golden Path 3: 작은 게임 제작 검증

```text
Personal Alpha 단계에서 작은 게임 repository 준비
→ AI가 기능 추가와 버그 수정
→ 테스트 실행과 artifact 검토
→ review와 merge
```

작은 게임 제작은 첫 구현 검증이 아니다. Golden Path 1과 2가 통과한 뒤 Personal Alpha에서 제품 흐름을 검증한다.

## Remote Test Runner 구현 순서

Remote Test Runner는 [[07 ADR/ADR-0012 Remote Test Runner Worker Capability]]에 따라 MVP 전체에 포함하지만 첫 Slice가 아니다.

```text
1. Local Worker Golden Path
2. Codex/Antigravity CLI Adapter integration
3. Remote Test Runner Node registration
4. Tailscale 또는 사설 네트워크 연결
5. Git checkout 기반 test/build/lint
6. Artifact upload
7. Worker Report
8. Owner artifact review
```

Local Worker path가 먼저 Task Attempt, Report, ArtifactRef와 review 흐름을 제공해야 Remote Runner가 이를 재사용할 수 있다.

## Desktop App Shell 구현 순서

1차 MVP는 browser-accessible Desktop-ready Web UI로 구현한다. 장기 제품 목표인 Desktop App Shell은 후속 단계다.

Web UI를 Desktop Shell에서 재사용할 수 있게 만들되 Shell technology, packaging, auto update, OS keychain과 local IPC는 이 ADR에서 확정하지 않는다.

## 인증 구현 범위

1차 Slice는 [[07 ADR/ADR-0011 Personal Runtime, Account, Device, and Session Model]]의 다음 로컬 인증만 포함한다.

- Web UI pairing code
- opaque Session token과 token hash 저장
- Device와 Session 분리
- Device/Session 조회와 폐기

중앙 account 로그인, 회원가입, OAuth와 다중 기기 동기화는 후순위다.

## 포함 범위

### 1차 MVP Slice

- 새 v2 skeleton과 Public repository hygiene
- Monorepo structure
- FastAPI server와 React/Vite Web UI skeleton
- SQLite migration baseline과 Local Runtime app data directory
- Web UI pairing/Session
- Project/Repository와 Git dirty check
- Conversation, Message와 Agent Run 기록
- 최소 Tool Registry와 Tool Call Envelope
- Work Item, Task, Task Attempt와 Worker claim/lease
- Mock/Manual Adapter
- Git worktree, 작업 branch와 Attempt 자동 commit
- ArtifactRef와 Owner review
- 사용자 승인 후 squash merge
- Runtime/Audit Event

### MVP 전체의 후반 Slice

- Codex CLI Owner Adapter
- Antigravity CLI Worker Adapter
- Remote Test Runner Node와 Runner Agent
- Tailscale 또는 사설 네트워크 후보
- 원격 test/build/lint와 Worker Report
- Owner artifact review

## 제외 범위

- v1 repository fork 또는 코드 직접 복사
- Discord interface와 동기식 Owner CLI 구조
- Desktop App Shell packaging, OS keychain, auto update와 local IPC
- 중앙 account 로그인, 회원가입/OAuth와 다중 기기 동기화
- Team Mode Central Authority 구현
- Approval Group, Merge Coordinator와 팀 공유 Worker Pool
- Remote AI Worker Node
- Enterprise policy engine
- public internet exposure와 relay server
- 자동 scaling

## 구현 저장소 생성 후 첫 skeleton 요구사항

후속 작업에서 `ai-development-platform`을 만들 때 최소 다음 파일을 포함한다.

```text
README.md
AGENTS.md
.gitignore
.env.example
apps/server/README.md
apps/web/README.md
apps/runner-agent/README.md
packages/shared-contracts/README.md
packages/cli-adapters/README.md
scripts/dev.ps1
scripts/dev.sh
docs/README.md
```

초기 검증 명령 후보:

```text
pnpm install
pnpm -C apps/web dev

uv sync
uv run fastapi dev apps/server/src/main.py
```

정확한 command와 source layout은 skeleton 구현 시 조정할 수 있다.

## 장점

- v1 제약과 새 설계를 분리한다.
- 핵심 상태 머신과 Git pipeline을 외부 AI CLI보다 먼저 검증한다.
- Monorepo에서 contracts, server와 UI를 빠르게 함께 변경할 수 있다.
- Mock/Manual Adapter로 외부 CLI 장애가 구현 전체를 막지 않는다.
- Public repository hygiene와 runtime data 경계를 처음부터 적용한다.

## 단점

- 새 skeleton과 migration을 처음부터 작성해야 한다.
- Monorepo tooling과 Python/TypeScript contract 동기화가 필요하다.
- 목표 CLI Adapter와 Remote Runner가 첫 Slice 뒤로 밀린다.
- Ubuntu primary와 Windows compatibility를 함께 검증해야 한다.

## 후속 과제

- 실제 `ai-development-platform` Public repository 생성 시점
- skeleton 생성 prompt와 source layout
- 첫 Alembic migration DDL
- 최소 Tool subset과 input/output schema
- Codex/Antigravity non-interactive 실행, auth와 interactive prompt 처리
- stdout/stderr structured parsing
- Process Runner와 OS Path Resolver 상세 계약
- app data directory 위치
- branch/worktree naming rule
- Local Worker sandboxing
- Remote Test Runner network protocol
- Desktop App Shell 기술
- 중앙 account와 sync

## 관련 문서

- [[01 Product/Personal Mode MVP]]
- [[02 Architecture/System Context]]
- [[02 Architecture/Data Ownership]]
- [[03 Domain Model/Domain Model]]
- [[05 Database/Database Strategy]]
- [[09 Roadmap/Personal Mode MVP Roadmap]]
- [[10 Open Questions/Open Questions]]

