---
type: product
status: draft
updated: 2026-06-22
---

# Personal Mode MVP

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]
- [[07 ADR/ADR-0011 Personal Runtime, Account, Device, and Session Model]]
- [[07 ADR/ADR-0012 Remote Test Runner Worker Capability]]
- [[07 ADR/ADR-0013 MVP Implementation Slice and Repository Strategy]]
- [[07 ADR/ADR-0016 Initial Personal Mode MVP Execution Policy]]
- [[07 ADR/ADR-0017 Failure Recovery and Worktree Resume Policy]]
- [[07 ADR/ADR-0018 Personal Mode Approval Modes, Grants, and Autonomy Profiles]]

## 대상 사용자

- 단일 로컬 기기(PC/Laptop)에서 개발을 진행하는 1인 개발자 또는 개인 프로젝트 진행자.
- 안전하고 격리된 환경에서 AI에게 코딩을 맡기고 싶지만, 언제든 개입하고 검토할 수 있는 주도권을 원하는 사용자.

## Core Philosophy
- Initial MVP is the first checkpoint where Owner and Worker operate together like an agentic Codex-style app with Task structure.
- User interacts with Owner, not Worker.
- Owner can act autonomously within approval mode and project working scope.
- Worker executes Task only.
- Real AI Worker path such as AGY is an important checkpoint, not necessarily the only MVP core dependency.
- Initial MVP uses Codex CLI as the first Owner Runtime Provider. The product must remain open to API-based and CLI-based Owner runtimes.

## Initial MVP completion target
The initial Personal Mode MVP is considered achieved when:
- Owner and Worker can operate together through Task structure.
- C-level UI surfaces are available: Project/Repository, Conversation/Owner, Work Item/Task, Task Attempt, WorkerRun, Worktree, Diff, Artifact, Logs, Approval, Settings summary.
- A controlled real AI Worker path, such as AGY, can complete at least one happy path under Owner control.
- Failure/retry/recovery can return to Owner conversation.
- The product owner can dogfood the system by pausing platform development and using Personal Mode to build a small game project.

## 해결하려는 문제

사용자는 AI에게 개발 작업을 맡기고 싶지만, 코드 변경, 테스트, diff 검토, 병합 승인, 로그와 아티팩트 확인이 흩어져 있으면 실제 프로젝트에 쓰기 어렵다.

개인 모드 MVP는 Desktop App Shell에 재사용 가능한 Web UI에서 Owner 대화, Task 계획, Worktree 기반 Worker 실행, 테스트와 Diff 검토, 정책에 따른 승인 후 병합까지 이어지는 로컬 개발 루프를 제공한다.

## 구현 저장소와 첫 Slice

v2는 v1을 수정하거나 fork하지 않고 새 Public Monorepo `ai-development-platform`에서 구현한다. 이 설계 저장소 `ai-development-platform-design`은 결정 원본이며 `ai-game-company-server`는 참고 구현일 뿐이다.

Public repository에는 secret, API key, `.env`, local SQLite DB, 사용자 Project, artifact, worktree, runtime log와 local credential을 commit하지 않는다. 강한 `.gitignore`와 commit 가능한 `.env.example`을 skeleton에 포함한다.

첫 구현 Slice는 Mock/Manual Adapter로 Local Worker Golden Path를 완성한다. Task → Attempt → Worktree → Commit → Owner Review → 사용자 승인 → Squash Merge를 먼저 통과한 뒤 Codex CLI Owner와 Antigravity CLI Worker를 연결한다.

Codex와 Antigravity는 Owner/Worker Identity가 아니라 교체 가능한 외부 Agent Process Adapter다. Mock과 Manual Adapter는 개발·pipeline 검증·복구용이며 목표 Adapter를 대체하지 않는다.

## 대표 사용 시나리오

1. Windows 또는 Linux에서 App-managed Local Runtime을 시작한다.
2. 초기 Desktop-ready Web UI를 연다.
3. 1회용 pairing code로 브라우저 Device와 Session을 연결한다.
4. 기존 Git 프로젝트를 가져오고 primary repository를 확인한다.
5. Owner에게 개발 요청을 한다.
6. Owner가 Task를 계획한다.
7. Generic Worker가 Task Attempt별 Worktree와 작업 브랜치에서 작업하고 결과를 자동 commit한다.
8. 테스트와 Diff를 생성한다.
9. 사용자가 결과를 검토한다.
10. 승인 후 하나의 squash commit으로 기본 브랜치에 병합한다.

## 최초 설치 후 흐름

첫 설치 시 사용자 가입은 하지 않는다. App-managed Local Runtime이 local user 한 명을 자동 생성한다.

이 local user는 개인 Workspace Admin, 모든 개인 프로젝트 소유자, 개인 Owner의 권한 위임자, 연결된 Device 승인자다.

향후 중앙 account 연결과 다중 사용자 기능을 위해 local user와 future account를 분리하고 nullable `account_id`, Device·Session 관계, 프로젝트 소유권과 권한 구조를 유지한다. MVP에서는 중앙 account 로그인, 회원가입과 다중 기기 동기화를 구현하지 않는다.

## Windows 또는 Linux Local Runtime 개념

App-managed Local Runtime은 Windows와 Linux 지원을 목표로 설계한다.

Ubuntu Linux는 첫 개발, 통합 테스트와 실제 운영 기준으로 사용하는 Ubuntu reference environment다. 이는 Ubuntu만 지원한다는 의미가 아니다.

macOS는 첫 MVP의 공식 지원 범위에 포함하지 않는다.

## 브라우저 연결

제품의 공식 장기 UI 목표는 Desktop App이다. 초기 MVP는 빠른 개발을 위해 App-managed Local Runtime이 제공하는 browser-accessible, Desktop-ready Web UI로 시작한다.

초기 Web UI는 향후 Desktop App Shell 내부에서 재사용할 수 있어야 한다. localhost HTTP API는 브라우저 전용 API가 아니라 UI Shell과 Local Runtime을 연결하는 첫 local transport다. Desktop App에서는 같은 HTTP API 또는 후속 local IPC를 사용할 수 있다.

사용자는 초기 Web UI를 Local Runtime에 연결해 앱을 사용한다. Web UI는 Local Runtime이 발급한 10분 만료, 1회용 pairing code를 입력하면 Device와 Session을 발급받는다.

대표 연결:

`Desktop-ready Web UI → localhost HTTP API → App-managed Local Runtime`

Session은 opaque random token을 사용하고 SQLite에는 token hash만 저장한다. Web UI는 HttpOnly, SameSite cookie를 권장하며 민감 token을 localStorage에 저장하지 않는다.

Tailscale은 앱 접속의 기본 전제가 아니며 MVP 사용을 위해 설치할 필요가 없다. Remote Test Runner 연결을 위한 선택적 사설 네트워크 후보이며 LAN이나 SSH tunnel 등으로 대체할 수 있어야 한다.

## 프로젝트 가져오기

첫 프로젝트 추가 기능은 Local Runtime이 접근할 수 있는 기존 Git 저장소 가져오기다.

Project는 여러 `ProjectRepository`를 가질 수 있지만 첫 실행 흐름은 primary repository 중심으로 제공한다. 다중 repository 모델은 유지하되 cross-repository atomic merge와 multi-repo merge orchestration은 MVP 범위에서 제외한다.

사용자는 허용된 프로젝트 루트 아래의 저장소를 선택하거나 경로를 입력한다.

다음 단계는 Git URL Clone이다. 빈 프로젝트 생성은 그 다음 단계다.

프로젝트 가져오기에서는 경로 정규화, Path traversal 검사, symlink 또는 junction 검사, 저장소 유효성 검사, dirty 상태 확인, Canonical Path 비교를 수행한다.

## Owner와 대화

사용자는 웹 UI에서 Owner에게 개발 요청을 입력한다.

Owner는 Conversation과 Agent Run을 만들고, 필요한 프로젝트 맥락과 권한 정책을 조합해 작업 계획을 세운다.

Owner는 Internal Tool Contract를 통해서만 Local Control Plane의 상태 변경 기능을 호출한다. 플랫폼 Tool은 일반 검색이나 코드 작성 능력의 중복 구현이 아니라 Task, Worker, Git, 승인과 실행 기록처럼 플랫폼이 소유한 상태와 부작용을 통제하는 서버 기능이다.

- User-to-Owner communication, Task-to-Worker execution separation is the MVP baseline.
- The user does not operate the system through command keywords. The user talks to Owner. Owner interprets intent and uses platform tools to create Tasks and manage execution. (See ADR-0019)
- General free-form Worker UI (e.g., chatting directly with a Worker) is not part of MVP.

## Worker 실행

첫 MVP의 Worker는 하나의 Generic Development Worker다.

Worker는 사용자의 기본 작업 디렉터리를 직접 수정하지 않는다. Task Attempt별 Git Worktree와 작업 브랜치 안에서만 파일을 수정한다.

Worker는 Owner가 만든 Task 없이 실행하지 않는다. 작업 결과는 작업 브랜치에 자동 commit하며, Worker가 기본 브랜치에 직접 병합할 수는 없다. 대상 repository가 dirty이면 읽기와 분석은 허용하되 Worker 작업 시작은 차단한다.

Worker는 Task Attempt를 claim하고 유효한 lease를 가진 동안 실행하며 heartbeat로 lease를 갱신한다. 로그, diff, 테스트 결과와 Worker report 같은 큰 증거물은 Artifact Store에 두고 SQLite의 `artifact_refs`로 연결한다.

- Owner-led Worker capability is proven by AGY Alpha. AGY Worker Alpha는 이러한 분리된 실행 구조를 검증하는 첫 번째 구현이다.

## Remote Test Runner

Remote Test Runner는 Personal Mode MVP에 포함하는 좁은 범위의 Worker Capability다. Owner는 Runner를 직접 조작하지 않고 테스트가 필요한 Task를 만든다. Worker가 Task Attempt를 claim한 뒤 등록된 Runner에서 test, build와 lint를 실행한다.

Remote Test Runner는 Local Worker Golden Path와 목표 CLI Adapter 통합 뒤에 구현한다.

Runner는 판단 주체나 AI Worker가 아니며 코드를 수정하거나 commit, merge, 배포하지 않는다. Git repository를 clone/fetch하고 지정된 commit을 checkout한 임시 복사본에서만 실행한다.

로그, screenshot, report와 파일 산출물은 Main App Artifact Store에 업로드하고 SQLite에는 `artifact_ref`를 저장한다. Worker가 원본 artifact를 분석해 Worker Report를 작성하며, Owner와 UI는 필요하면 artifact Tool을 통해 원본을 열람할 수 있다.

## Diff와 테스트 검토

Worker 실행 후 UI는 다음을 보여준다.

- Task 목표
- Owner 결과 요약
- 변경 파일
- Git diff
- 테스트 결과
- 린트 결과
- 실행한 명령
- 실패하거나 건너뛴 검사
- 위험 등급
- 병합 대상 브랜치
- 되돌리기 방법
- 운영체제 특이 경고

## 병합 승인

기본 `Allow Local Work`에서는 기본 브랜치 또는 사용자의 실제 작업 브랜치에 반영하기 전에 사용자가 승인한다.

Owner가 diff, 테스트 증거, scope 위반, 위험도와 stale base commit을 검토하고 승인 정책을 통과한 결과만 하나의 squash commit으로 반영한다.

승인 필요 여부는 Owner Grant와 Autonomy Profile에 따라 평가한다. 사용자가 특정 프로젝트, repository와 risk 범위에 더 높은 권한을 명시하면 자동 squash merge를 허용할 수 있지만, Owner는 정책을 우회하거나 자신의 권한을 확대할 수 없고 R4 행동은 자동 실행할 수 없다.

사용자 선택:

- 승인하고 병합
- 거부
- Owner에게 수정 요청
- 작업 결과 보관
- Task 취소

## 권한 설정

개인 모드의 기본 자율성 프로필은 `Allow Local Work`다.

MVP에서 최소 구현할 프로필:

- Confirm Every Change
- Allow Local Work

Trusted Owner와 Autonomous는 후속 구현할 수 있도록 데이터 모델과 UI 확장성을 유지한다.

단순한 `full_access` Boolean을 사용하지 않고 [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]의 Owner Grant와 위험 정책을 따른다.

## UI Shell과 App-managed Local Runtime

UI Shell은 사용자가 앱을 이용하는 제품 표면이다. 첫 UI Shell은 브라우저이고 장기적으로 Desktop App Shell로 확장한다.

UI Shell 역할:

- 브라우저에서 웹 UI 사용
- Owner와 대화
- 작업 진행 확인
- Diff와 테스트 결과 검토
- 승인과 거부
- 설정 변경
- 연결된 장치 관리

UI Shell은 Local Transport를 우회해 SQLite, Git이나 파일시스템을 직접 변경하지 않는다.

App-managed Local Runtime 역할:

- 웹 프론트엔드 정적 파일 제공
- Local Control Plane
- Owner Supervisor
- Agent Run Engine
- CLI Model Adapter
- Worker Supervisor
- Generic Development Worker
- SQLite
- Git Repository와 Worktree
- 로그와 아티팩트
- 백그라운드 작업

`Primary Personal Server`는 내부 배치 용어로 유지할 수 있지만 사용자에게 별도 서버 제품으로 설명하지 않는다. 장기적으로 Desktop App이 Local Runtime을 시작하고 관리하며, 앱 창을 닫아도 작업이 이어질 수 있도록 Background Service로 분리할 수 있다. 설치 방식은 후속 설계다.

## Device와 Session 관리

Device는 등록된 앱·브라우저 단위이고 Session은 해당 Device의 현재 인증 상태다. 사용자는 설정에서 Device와 Session 목록, device name과 type, 마지막 접속을 보고 특정 Device, 특정 Session 또는 모든 Session을 폐기할 수 있어야 한다.

모든 Session을 잃은 경우 Local Runtime의 CLI 또는 console에서 새 pairing code를 재발급한다. SQLite 직접 수정은 공식 복구 절차가 아니다.

## MVP에 포함되는 기능

- 웹 UI와 API
- Desktop-ready Web UI와 localhost HTTP API
- Internal Tool Contract와 Tool Registry
- Tool Boundary와 Local Policy Engine
- 단일 로컬 사용자 자동 생성
- 일회용 연결 코드 기반 장치 연결
- Device와 Session 분리, 목록과 폐기
- opaque Session token과 token hash 저장
- 기존 Git 저장소 가져오기
- Owner Conversation
- Agent Run과 Tool Call
- CLI-first Model Adapter
- Generic Development Worker
- Worker lease/claim
- Git Worktree 격리
- ProjectRepository와 primary repository 중심 실행
- Worker 작업 브랜치 자동 commit
- 테스트와 Diff 증거
- Confirm Every Change와 Allow Local Work
- Owner Grant와 Autonomy Profile 평가 후 squash merge
- SQLite 기반 실행 상태 저장
- 로그와 아티팩트 참조
- Artifact Store와 `artifact_refs`
- Remote Test Runner Worker Capability
- 등록된 Test Runner Node와 heartbeat/status
- Git checkout 기반 test/build/lint
- Test Runner artifact upload와 Worker Report
- Public v2 Monorepo와 repository hygiene
- Local Worker Golden Path
- Mock/Manual Adapter와 목표 Codex/Antigravity Adapter
- Emergency Stop 기본 흐름

## MVP에 포함되지 않는 기능

- 사용자 가입
- 이메일 인증
- 비밀번호 재설정
- 다중 사용자
- 중앙 Authority
- 팀 Workspace
- 팀 멤버십
- Approval Group
- 팀 Merge Queue
- PostgreSQL
- 여러 Worker Host
- 일반 Worker Task의 여러 실행 서버 분산
- 모바일 전용 앱
- Desktop App Shell 패키징과 local IPC
- 고가용성
- Enterprise SSO
- 결제와 사용량 청구
- 전문 Worker 분리
- 중앙 Coordination AI
- macOS 공식 지원
- 운영체제별 자동 업데이트 시스템
- OS keychain 연동
- 중앙 account 로그인, 회원가입과 다중 기기 동기화
- Remote AI Worker Node
- relay server와 public internet Runner exposure
- 팀 공유 Test Runner Pool과 고급 scheduling

작은 게임 제작은 첫 구현 Slice가 아니라 Golden Path 1·2 이후 Personal Alpha에서 수행하는 검증 시나리오다.

## 성공 기준

- Ubuntu reference environment에서 실제 Git 저장소를 가져와 작업을 완료한다.
- Windows에서도 App-managed Local Runtime이 실행되고 같은 Web UI/API를 제공한다.
- 브라우저 연결이 끊겨도 승인 대기 전까지 작업 상태가 SQLite에 남는다.
- Worker가 기본 작업 디렉터리를 직접 수정하지 않고 Worktree에서만 작업한다.
- 사용자가 diff와 테스트 결과를 보고 병합 여부를 결정할 수 있다.
- 사용자 가입 없이도 user_id, 세션, 권한 구조가 유지된다.
- Tailscale 없이 기본 Personal Mode 흐름을 완료할 수 있다.
