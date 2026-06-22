# Personal Mode MVP Roadmap

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]
- [[07 ADR/ADR-0011 Personal Runtime, Account, Device, and Session Model]]
- [[07 ADR/ADR-0012 Remote Test Runner Worker Capability]]
- [[07 ADR/ADR-0013 MVP Implementation Slice and Repository Strategy]]

## Phase 0 - Public v2 Monorepo and Skeleton

- 새 Public `ai-development-platform` 저장소 생성
- `apps/server`, `apps/web`, `apps/runner-agent` 구조
- `packages/shared-contracts`, `packages/cli-adapters` 구조
- 강한 `.gitignore`와 `.env.example`
- uv와 pnpm 개발 환경
- Python/FastAPI 기본 구조
- React/TypeScript/Vite 기본 구조
- Desktop App Shell 재사용을 고려한 Desktop-ready Web UI 구조
- localhost HTTP API와 Internal Tool Contract 경계
- Windows와 Linux를 고려한 설정 시스템
- 운영체제 Path Resolver
- Process Runner 추상화
- SQLite + SQLAlchemy 2 + Alembic baseline
- 테스트 기반
- Ubuntu CI 또는 reference test
- Windows CI 또는 compatibility test

완료 조건:

- 서버와 프론트엔드 skeleton이 로컬에서 실행된다.
- 설정, 경로, 프로세스 실행 경계가 운영체제 중립 인터페이스로 분리된다.
- Ubuntu Desktop primary와 Windows compatibility 개발 환경이 재현 가능하다.

주요 위험:

- 초기부터 Ubuntu 경로와 Shell 문자열에 묶이는 것
- Windows 실행 파일 탐색과 경로 길이 문제를 늦게 발견하는 것

## Phase 1 - Local Control Plane

- 단일 로컬 사용자 자동 생성
- local user와 nullable 중앙 account 연결 모델
- 연결된 장치와 세션
- Web UI 1회용 pairing code
- Device와 Session 목록·폐기
- opaque Session token과 token hash 저장
- Project와 ProjectRepository 모델
- 기존 Git 저장소 가져오기
- 기본 API
- 이벤트와 감사 기록
- 허용 프로젝트 루트
- 운영체제별 경로 검증

완료 조건:

- 일회용 코드로 브라우저 세션을 연결할 수 있다.
- Tailscale 없이 Local Runtime에 Web UI를 연결하고 Device·Session을 폐기할 수 있다.
- 허용 프로젝트 루트 아래의 Git 저장소를 가져올 수 있다.

주요 위험:

- 장치 세션을 사용자 권한과 혼합하는 것
- 경로 정규화, symlink, junction 검사를 빠뜨리는 것

## Phase 2 - Owner Runtime

- Conversation과 Message
- Owner Supervisor
- Agent Run과 Step
- Agent Process Adapter interface와 Mock Adapter
- 운영체제별 CLI 실행 파일 탐색
- 출력 스트리밍
- 취소와 복구

완료 조건:

- 사용자가 Owner에게 요청하면 Agent Run이 생성되고 상태가 SQLite에 저장된다.
- Adapter 출력이 UI에 스트리밍되고 재시작 후 상태를 복구할 수 있다.

주요 위험:

- Adapter 출력 문자열을 공식 도메인 상태로 직접 사용하는 것
- Mock 결과와 저장된 공식 실행 상태를 혼합하는 것

## Phase 3 - Worker Execution

- Generic Development Worker
- Task와 Task Attempt
- Worker Supervisor
- Git Worktree
- repository별 dirty check
- 작업 브랜치 자동 commit
- Windows와 Linux Worktree 테스트
- 안전한 명령 실행
- 테스트와 Diff 증거
- Mock Worker Adapter
- Manual Worker Adapter

완료 조건:

- Worker가 별도 Worktree에서 파일을 수정하고 diff와 테스트 결과를 남긴다.
- 기본 작업 디렉터리의 미커밋 변경을 덮어쓰지 않는다.
- README 수정 Golden Path를 외부 AI CLI 없이 완료할 수 있다.

주요 위험:

- Worktree 정리 실패
- 파일 잠금, 대소문자, 줄바꿈, 실행 권한 차이로 인한 운영체제별 오류

## Phase 4 - Approval and Merge

- R0~R4 Policy Engine
- Confirm Every Change
- Allow Local Work
- Approval UI
- Owner Grant
- Owner Grant와 Autonomy Profile 기반 merge 승인 평가
- Owner 검토와 squash merge
- stale 승인

완료 조건:

- 기본 브랜치 반영 전 사용자가 diff, 테스트, 위험 등급을 확인하고 승인할 수 있다.
- 승인 대상 버전이 바뀌면 stale 처리된다.

주요 위험:

- R4 자동 실행 허용
- 단순 `full_access` Boolean으로 권한을 표현하는 것

## Phase 4A - Target CLI Adapter Integration

- Codex CLI Owner Adapter
- Antigravity CLI Worker Adapter
- Adapter contract test
- non-interactive 실행, auth와 prompt 실험
- stdout/stderr와 실패 정규화

완료 조건:

- Mock/Manual Adapter로 검증된 Local Worker Golden Path를 목표 CLI Adapter로 다시 완료한다.

주요 위험:

- CLI 인증 만료, interactive prompt와 process 종료를 Agent Run 실패와 구분하지 못하는 것
- stdout/stderr 문자열을 구조화된 결과 없이 도메인 상태로 사용하는 것

## Phase 4B - Remote Test Runner

- Remote Test Runner Node 등록
- Tailscale 또는 같은 사설 네트워크 연결 후보
- Worker Capability 기반 원격 test/build/lint
- Git commit checkout 기반 실행
- Artifact Store upload와 Worker Report
- Owner artifact 원본 검토

완료 조건:

- Worker가 Task Attempt 범위에서 등록된 Runner를 사용하고 Owner가 Worker Report와 원본 artifact를 검토할 수 있다.

## Phase 5 - Web Experience

- browser-accessible Web UI 우선 구현
- Desktop App Shell에서 재사용 가능한 UI 구성
- 프로젝트 화면
- Owner 채팅
- Task 진행
- 로그와 스트리밍
- Diff Viewer
- 승인 카드
- 설정
- 연결된 장치
- 운영체제와 CLI 상태 표시

완료 조건:

- 브라우저 하나에서 프로젝트 가져오기, Owner 요청, 작업 진행, diff 검토, 승인까지 이어진다.
- UI Shell은 공식 상태 원본이 아니며 재접속 시 Local Runtime SQLite 상태로 복구된다.

주요 위험:

- 긴 로그와 diff 표시 성능
- 승인 UI가 위험과 되돌리기 방법을 충분히 설명하지 못하는 것

## Phase 6 - Personal Alpha

- 실제 저장소 테스트
- Ubuntu reference environment 검증
- Windows 설치와 실행 검증
- 재시작 복구
- CLI 로그인 만료
- Worktree 정리
- 백업
- 보안 점검
- 설치 문서
- 운영 테스트
- Golden Path 1과 2 반복 검증
- 작은 게임 제작 Golden Path 3 검증

완료 조건:

- 실제 개인 저장소에서 반복적으로 작업을 완료한다.
- Ubuntu reference environment와 Windows 실행 경로에서 핵심 흐름이 검증된다.

주요 위험:

- SQLite 백업과 로그 보존 정책 미정
- Windows Service 또는 사용자 세션 기반 실행 선택 지연
- pairing code와 Session 복구 UX 미완성

## Deferred

- 팀 모드와 중앙 Authority
- PostgreSQL
- 여러 Worker Host
- 전문 Worker 분리
- 데스크톱 패키징
- local IPC
- OS keychain 연동
- 자동 업데이트
- 중앙 account 로그인과 다중 기기 동기화
- Background Service 설치 방식
- Remote AI Worker Node
- macOS 공식 지원
