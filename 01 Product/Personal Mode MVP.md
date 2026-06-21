---
type: product
status: draft
updated: 2026-06-21
---

# Personal Mode MVP

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]

## 대상 사용자

개인 모드 MVP의 대상 사용자는 자신의 Windows 또는 Linux 컴퓨터에 Primary Personal Server를 실행하고, 브라우저에서 Owner와 대화하며 개인 Git 프로젝트를 개발하려는 개발자다.

팀 초대, 중앙 Authority, 다중 사용자 협업보다 먼저 개인 개발 루프를 안정화한다.

## 해결하려는 문제

사용자는 AI에게 개발 작업을 맡기고 싶지만, 코드 변경, 테스트, diff 검토, 병합 승인, 로그와 아티팩트 확인이 흩어져 있으면 실제 프로젝트에 쓰기 어렵다.

개인 모드 MVP는 하나의 웹 UI에서 Owner 대화, Task 계획, Worktree 기반 Worker 실행, 테스트와 Diff 검토, 승인 후 병합까지 이어지는 로컬 개발 루프를 제공한다.

## 대표 사용 시나리오

1. Windows 또는 Linux Primary Personal Server에서 서비스를 시작한다.
2. UI Client 장치를 Tailscale에 연결한다.
3. 일회용 코드로 브라우저를 연결한다.
4. 기존 Git 프로젝트를 가져오고 primary repository를 확인한다.
5. Owner에게 개발 요청을 한다.
6. Owner가 Task를 계획한다.
7. Generic Worker가 Task Attempt별 Worktree와 작업 브랜치에서 작업하고 결과를 자동 commit한다.
8. 테스트와 Diff를 생성한다.
9. 사용자가 결과를 검토한다.
10. 승인 후 하나의 squash commit으로 기본 브랜치에 병합한다.

## 최초 설치 후 흐름

첫 설치 시 사용자 가입은 하지 않는다. Primary Personal Server가 로컬 사용자 한 명을 자동 생성한다.

이 로컬 사용자는 개인 Workspace Admin, 모든 개인 프로젝트 소유자, 개인 Owner의 권한 위임자, 연결된 장치 승인자다.

향후 가입과 다중 사용자 기능을 위해 user 엔티티, user_id 참조, 장치와 사용자 관계, 프로젝트 소유권, 세션과 권한 구조는 유지한다.

## Windows 또는 Linux 서버 설치 개념

Primary Personal Server Runtime은 Windows와 Linux 지원을 목표로 설계한다.

Ubuntu Linux는 첫 개발, 통합 테스트와 실제 운영 기준으로 사용하는 Ubuntu reference environment다. 이는 Ubuntu만 지원한다는 의미가 아니다.

macOS는 첫 MVP의 공식 지원 범위에 포함하지 않는다.

## 브라우저 연결

첫 UI는 설치형 데스크톱 앱이 아니라 Primary Personal Server가 제공하는 웹 UI다.

사용자는 같은 컴퓨터의 브라우저 또는 Tailscale로 연결된 다른 개인 장치의 브라우저에서 접속한다.

대표 연결:

`작업 노트북 브라우저 → Tailscale → Primary Personal Server 웹 앱`

Tailscale 설치, 계정 가입과 로그인 자동화는 MVP 필수 범위가 아니다. 사용자가 Tailscale을 미리 설치하고 구성했다고 가정한다.

## 프로젝트 가져오기

첫 프로젝트 추가 기능은 Primary Personal Server에 이미 존재하는 Git 저장소 가져오기다.

Project는 여러 `ProjectRepository`를 가질 수 있지만 첫 실행 흐름은 primary repository 중심으로 제공한다. 다중 repository 모델은 유지하되 cross-repository atomic merge와 multi-repo merge orchestration은 MVP 범위에서 제외한다.

사용자는 허용된 프로젝트 루트 아래의 저장소를 선택하거나 경로를 입력한다.

다음 단계는 Git URL Clone이다. 빈 프로젝트 생성은 그 다음 단계다.

프로젝트 가져오기에서는 경로 정규화, Path traversal 검사, symlink 또는 junction 검사, 저장소 유효성 검사, dirty 상태 확인, Canonical Path 비교를 수행한다.

## Owner와 대화

사용자는 웹 UI에서 Owner에게 개발 요청을 입력한다.

Owner는 Conversation과 Agent Run을 만들고, 필요한 프로젝트 맥락과 권한 정책을 조합해 작업 계획을 세운다.

## Worker 실행

첫 MVP의 Worker는 하나의 Generic Development Worker다.

Worker는 사용자의 기본 작업 디렉터리를 직접 수정하지 않는다. Task Attempt별 Git Worktree와 작업 브랜치 안에서만 파일을 수정한다.

Worker는 Owner가 만든 Task 없이 실행하지 않는다. 작업 결과는 작업 브랜치에 자동 commit하며, Worker가 기본 브랜치에 직접 병합할 수는 없다. 대상 repository가 dirty이면 읽기와 분석은 허용하되 Worker 작업 시작은 차단한다.

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

기본 브랜치 또는 사용자의 실제 작업 브랜치에 반영하기 전에는 사용자가 승인한다.

Owner가 diff, 테스트 증거, scope 위반, 위험도와 stale base commit을 검토하고 승인 정책을 통과한 결과만 하나의 squash commit으로 반영한다.

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

## UI Client와 Primary Personal Server

UI Client는 사용자의 작업 노트북, 태블릿 또는 다른 개인 장치다.

UI Client 역할:

- 브라우저에서 웹 UI 사용
- Owner와 대화
- 작업 진행 확인
- Diff와 테스트 결과 검토
- 승인과 거부
- 설정 변경
- 연결된 장치 관리

UI Client에 프로젝트 코드, Owner Runtime 또는 주 SQLite DB를 필수로 저장하지 않는다.

Primary Personal Server 역할:

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

## MVP에 포함되는 기능

- 웹 UI와 API
- 단일 로컬 사용자 자동 생성
- 일회용 연결 코드 기반 장치 연결
- 기존 Git 저장소 가져오기
- Owner Conversation
- Agent Run과 Tool Call
- CLI-first Model Adapter
- Generic Development Worker
- Git Worktree 격리
- ProjectRepository와 primary repository 중심 실행
- Worker 작업 브랜치 자동 commit
- 테스트와 Diff 증거
- Confirm Every Change와 Allow Local Work
- 승인 후 squash merge
- SQLite 기반 실행 상태 저장
- 로그와 아티팩트 참조
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
- 여러 실행 서버의 작업 분산
- 모바일 전용 앱
- 데스크톱 패키징
- 자동 Tailscale 설치
- 고가용성
- Enterprise SSO
- 결제와 사용량 청구
- 전문 Worker 분리
- 중앙 Coordination AI
- macOS 공식 지원
- 운영체제별 자동 업데이트 시스템

## 성공 기준

- Ubuntu reference environment에서 실제 Git 저장소를 가져와 작업을 완료한다.
- Windows에서도 Primary Personal Server가 실행되고 같은 웹 UI/API를 제공한다.
- 브라우저 연결이 끊겨도 승인 대기 전까지 작업 상태가 SQLite에 남는다.
- Worker가 기본 작업 디렉터리를 직접 수정하지 않고 Worktree에서만 작업한다.
- 사용자가 diff와 테스트 결과를 보고 병합 여부를 결정할 수 있다.
- 사용자 가입 없이도 user_id, 세션, 권한 구조가 유지된다.
