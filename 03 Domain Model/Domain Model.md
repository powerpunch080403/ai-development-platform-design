# Domain Model

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]

## App Client

사용자가 보는 하나의 앱과 UI. 개인 프로젝트에서는 Local Control Plane에 연결하고, 팀 프로젝트에서는 Personal Node와 중앙 Authority에 연결한다.

개인 모드 MVP의 첫 App Client는 브라우저 기반 UI Client다.

## UI Client

사용자의 작업 노트북, 태블릿 또는 다른 개인 장치에서 브라우저로 접속하는 UI 실행 환경.

UI Client는 Owner와 대화하고, 작업 진행을 확인하고, Diff와 테스트 결과를 검토하고, 승인과 설정을 수행한다. UI Client는 프로젝트 코드, Owner Runtime 또는 주 SQLite DB의 필수 저장 위치가 아니다.

## Local Control Plane

개인 모드에서 개인 프로젝트 상태를 관리하는 로컬 제어 계층.

- Work Item과 Task 관리
- Task Attempt 관리
- Owner Runtime과 Worker Supervisor 조정
- 로컬 승인
- 로컬 병합
- 실행 로그 관리
- 실패 복구

## Personal Node

사용자별 실행 환경. Owner Supervisor, Conversation Store, Agent Run Engine, Worker Supervisor, Local Git Workspace, 로컬 SQLite, Inbox/Outbox, 로컬 실행 로그와 오프라인 작업 상태를 포함할 수 있다.

팀 모드의 Personal Node는 중앙 Authority와 연결되지만 중앙 DB의 복제본이나 동등한 Writer가 아니다.

`Node`는 내부 시스템 용어다. 사용자 화면에서는 가능한 한 `개인 서버`, `실행 서버`, `연결된 장치`, `이 컴퓨터` 같은 표현을 사용한다.

## Primary Personal Server

개인 모드 MVP에서 Local Control Plane, Owner Supervisor, Worker Supervisor, SQLite, Local Git Workspace, 웹 UI/API를 실행하는 사용자별 주 실행 환경.

Primary Personal Server Runtime은 Windows와 Linux 지원을 목표로 설계한다. Ubuntu Linux는 초기 구현과 검증을 위한 Ubuntu reference environment다.

## Runtime Platform Adapter

운영체제별 차이를 Runtime 핵심 로직에서 분리하는 Adapter.

- 경로 Resolver
- 명령 실행
- 실행 파일 확장자와 검색 경로 처리
- 서비스 설치와 백그라운드 실행
- 파일 잠금, 심볼릭 링크, 경로 길이, 줄바꿈, 실행 권한 차이 처리

## User

시스템을 사용하는 인간 계정.

개인 모드 MVP에서는 사용자 가입을 구현하지 않고 첫 설치 시 로컬 사용자 한 명을 자동 생성한다. 그래도 향후 사용자 가입과 다중 사용자 기능을 위해 user 엔티티, user_id 참조, 장치와 사용자 관계, 프로젝트 소유권, 세션과 권한 구조를 유지한다.

사용자 정보를 전역 상수나 하드코딩된 단일 ID로 처리하지 않는다.

## Reference Environment

초기 개발, 통합 테스트와 실제 운영 기준으로 사용하는 환경. 개인 모드 MVP의 Reference Environment는 Ubuntu Linux다.

Reference Environment는 유일한 지원 운영체제를 의미하지 않는다.

## Supported Operating System

제품 지원 목표에 포함되는 운영체제. 개인 모드 MVP의 Primary Personal Server Runtime은 Windows와 Linux 지원을 목표로 한다.

macOS는 첫 MVP의 공식 지원 범위에 포함하지 않는다.

## Connected Device

Primary Personal Server에 연결된 UI Client 장치 또는 브라우저 실행 환경. Tailscale에 연결되어 있더라도 자동으로 앱에 로그인되지 않으며, 최초 연결에는 일회용 연결 코드가 필요하다.

## Device Session

승인된 Connected Device 또는 브라우저 세션에 발급된 세션. 설정 화면에서 확인하고 폐기할 수 있어야 한다.

## Project Import

Primary Personal Server에 프로젝트를 추가하는 절차.

구현 순서:

1. 이미 존재하는 Git 저장소 가져오기
2. Git URL로 Clone
3. 빈 프로젝트 생성

## ProjectRepository

Project에 속한 Git Repository와 역할을 나타내는 엔티티. 하나의 Project는 여러 ProjectRepository를 가질 수 있으며 role, local_path, remote_url, default_branch와 status를 추적한다.

role 기본 후보는 `primary`, `supporting`, `docs`, `infra`, `unknown`이다. Personal Mode MVP의 첫 실행은 primary repository 중심이지만 Task, Task Attempt, Worktree, Branch와 Commit은 항상 대상 repository를 명시한다.

## Allowed Project Root

프로젝트 가져오기를 기본 허용하는 상위 경로. 설정된 Allowed Project Root 밖의 임의 경로는 기본적으로 가져오지 않는다.

Project Import는 경로 정규화, Path traversal 검사, symlink 또는 junction 검사, 저장소 유효성 검사, dirty 상태 확인, Canonical Path 비교를 수행한다.

## Platform Path Resolver

운영체제별 기본 데이터 경로와 프로젝트 경로를 결정하는 Adapter. `/home/...`, `/var/...`, `C:\Users\...`, `C:\ProgramData\...` 같은 경로를 핵심 코드에 고정하지 않는다.

## Process Runner

명령 실행 Adapter. Shell 문자열보다 executable, argument array, working directory, environment variables, timeout, cancellation token 구조를 사용한다.

Windows와 Linux의 실행 파일 확장자, 검색 경로, 파일 권한, 경로 길이와 잠금 차이를 처리한다.

## Service Adapter

운영체제별 서비스 설치와 백그라운드 실행을 분리하는 Adapter. Linux systemd, Windows Service, Windows 사용자 세션 기반 백그라운드 실행, 시작 프로그램 등록은 후보이며 구체적인 선택은 후속 설계로 남긴다.

## CLI Model Adapter

CLI 방식 Model Provider Adapter. Codex CLI Adapter, Gemini CLI Adapter, Generic Command Adapter 같은 구현을 둘 수 있으며, 이후 API Adapter를 추가할 수 있어야 한다.

CLI 계정 가입, 구독 상태 관리, 결제, OAuth 로그인, 사용량 구매는 MVP 내부 기능이 아니다.

## Generic Development Worker

첫 MVP에서 사용하는 단일 범용 개발 Worker. Owner가 만든 Task를 할당받아 프로젝트와 코드 읽기, 허용된 Worktree 안의 파일 수정, 안전한 명령 실행, 포맷터, 린터, 테스트, Git status와 diff 생성, 작업 브랜치 자동 commit, 결과 요약, 실패 보고를 수행한다.

Worker는 Task 없이 실행하거나 제품 목표와 작업 단위를 임의로 정의하지 않는다.

전문 Worker 분리는 안정화 후로 미룬다.

## Git Worktree

Task Attempt별 격리 작업 공간. 하나의 active Task Attempt는 하나의 active Git Worktree를 가지며, Worktree는 하나의 ProjectRepository에 속한다. Worker는 사용자의 기본 작업 디렉터리를 직접 수정하지 않고 별도 Worktree와 작업 브랜치에서 작업한다.

대상 repository가 dirty이면 등록과 읽기·분석은 가능하지만 Worker 작업 시작은 차단한다. 앱은 사용자의 변경을 자동 commit하거나 stash하지 않는다.

## CommitRef

Task Attempt의 base, 작업 브랜치 final, squash commit을 Git 객체와 연결하는 SQLite 참조. Commit 객체와 이력의 원본은 Git이다.

## MergeRecord

ProjectRepository와 Task Attempt, base commit, head commit과 squash commit을 연결하는 병합 기록. Owner 검토와 승인 정책 통과 여부를 추적한다.

## Merge Approval

Worker 결과를 기본 브랜치 또는 사용자의 실제 작업 브랜치에 반영하기 전 사용자가 수행하는 승인. UI는 Task 목표, Owner 결과 요약, 변경 파일, Git diff, 테스트와 린트 결과, 실행 명령, 위험 등급, 병합 대상 브랜치, 되돌리기 방법을 보여준다.

Worker는 기본 브랜치에 직접 병합하지 않는다. Owner가 결과와 stale base를 검토하고 Approval Policy와 Owner Grant를 통과한 뒤 사용자가 승인하면 하나의 squash commit으로 반영한다.

## Central Authority

팀 프로젝트의 공식 공유 상태를 관리하는 중앙 권위 계층.

- Project Membership
- 공유 Work Item과 Task
- Task Lease
- Scope Lock
- Approval Policy
- Decision Proposal
- Change Package
- Merge Coordinator
- Audit Event

초기 버전에서 필수 중앙 AI를 두지 않는다.

## Owner Runtime

사용자별 Personal Node에서 실행되는 Owner 실행 시스템. 하나의 영구 LLM 세션이나 하나의 무한 실행 프로세스가 아니라 Owner Supervisor, Conversation, Agent Run, Agent Run Step, Tool Call, Approval Interruption, Memory, Model Provider Adapter, Event Stream으로 구성된다.

## Owner Supervisor

Personal Node에서 실행되는 장기 서비스. 사용자 요청 수신, Conversation 관리, Agent Run 생성, 실행 Queue 관리, Run 중단·재개·취소·재시도, Approval 대기 관리, Worker 작업 제출과 결과 대기, 오류 복구, 진행 이벤트 발행, 모델과 도구 정책 적용을 담당한다.

Owner Supervisor 자체는 하나의 계속 이어지는 AI 대화가 아니다.

## Conversation

사용자와 개인 Owner 사이의 장기 대화 단위. Project에 속하거나 프로젝트 없는 일반 대화로 존재할 수 있다. 여러 Agent Run을 포함할 수 있으며 Message 기록과 Agent Run 실행 상태를 분리한다.

일반 대화 중 사용자가 수정, 생성, 테스트, 분석 또는 병합 준비를 요청하면 Owner는 Agent Run을 만들고 필요한 Work Item이나 Task를 Tool Call로 생성한다.

후보 필드:

- conversation_id
- user_id
- project_id
- title
- status
- created_at
- updated_at

## Message

Conversation에 속한 대화 메시지.

후보 필드:

- message_id
- conversation_id
- actor_type
- actor_id
- content
- created_at
- related_run_id
- attachment_refs

## Agent Run

하나의 명확한 사용자 요청 또는 시스템 목적을 처리하는 제한된 실행 단위. Conversation 전체와 같지 않으며 고유한 입력, 상태, 권한, 모델 설정, 결과와 실행 이력을 가진다.

ADR-0009에서 확정한 상태:

- queued
- preparing_context
- running_model
- executing_tool
- waiting_for_approval
- waiting_for_user
- waiting_for_worker
- reviewing_worker_result
- retry_scheduled
- completed
- failed
- cancelled

구체적인 전이와 복구 규칙은 [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]을 따른다.

## Agent Run Step

Agent Run을 구성하는 순서 있는 실행 단계. 예: context_load, model_call, tool_call, worker_dispatch, worker_result_review, approval_request, decision_record, final_response.

Run 재개 시 이미 성공한 외부 작업을 무조건 다시 실행하지 않도록 Step 결과를 기록한다.

## Tool Call

등록된 Tool을 실행하는 기록 단위. LLM은 시스템 상태나 파일을 직접 임의 변경하지 않고, 외부 부작용은 Tool Call을 통해서만 실행한다.

외부 상태를 변경하는 Tool Call에는 idempotency key를 사용한다.

Owner LLM의 SQLite, Git, 파일시스템과 Worker 직접 조작은 금지한다. Local Control Plane 또는 Personal Server는 모든 Tool Call 전에 권한, 위험도, Approval Policy와 Owner Grant를 평가하고 Runtime Event와 Audit Event를 기록한다.

## Run State

현재 Agent Run을 재개하기 위한 실행 상태. 현재 Step, Tool Call 결과, Approval 상태, Worker 대기 상태, 일시적인 작업 Context를 포함한다.

Conversation History 및 Long-term Memory와 분리한다.

## Context Builder

Agent Run 시작 시 필요한 정보만 조합하는 구성 요소. 현재 사용자 요청, 최근 관련 메시지, 프로젝트 공식 결정, 관련 Work Item과 Task, 필요한 코드 또는 문서 요약, 사용자 및 프로젝트 Memory, 사용 가능한 Tool, 현재 권한 정책을 선택해 모델 입력을 만든다.

## Model Provider Adapter

Owner Runtime이 특정 AI 공급자나 모델에 결합하지 않도록 하는 교체 경계. Run마다 provider, model, prompt_version, tool_definition_version, runtime_version을 기록한다.

구체적인 모델 선택 및 fallback 정책은 아직 확정하지 않는다.

## Approval Interruption

민감 Tool Call이 승인을 요구할 때 Agent Run을 실패시키지 않고 대기 상태로 전환하는 중단 기록. 승인 대상 Tool Call, 요청한 권한, 인수 요약, 위험도, 승인 만료 시각, 재개에 필요한 버전 정보를 저장한다.

승인 또는 거부 후 같은 Run을 이어서 실행한다.

## Risk Level

Tool Call 또는 Action의 위험 등급. 초기 등급은 R0 관찰, R1 로컬·가역 변경, R2 공유 프로젝트 변경 제안, R3 고영향 행동, R4 치명적·보안 관리 행동이다.

수치 임계값은 아직 확정하지 않는다.

## Autonomy Profile

사용자가 Owner 자율성 기본값을 고르는 정책 템플릿. Confirm Every Change, Allow Local Work, Trusted Owner, Autonomous를 제공한다.

프로필은 편의를 위한 템플릿이며 실제 권한은 Owner Grant로 표현한다.

## Owner Grant

사용자가 Owner에게 위임한 범위 있는 권한. allowed_actions, allowed_tools, allowed_scopes, allowed_environments, maximum_risk_level, budget_limit, valid_from, expires_at, allow_background_execution, allow_external_network, allow_secret_use 등을 가질 수 있다.

Owner는 Grant를 생성, 확대, 연장하거나 취소 철회 상태를 변경할 수 없다.

## Approval Request

승인이 필요한 Action에 대해 생성되는 요청 기록. 요청한 행동, 대상 프로젝트와 환경, 대상 리소스, 변경 범위 요약, 위험 등급, 위험 산정 이유, 예상 영향, 되돌리기 방법, 비용 또는 외부 영향, 사용될 권한, 요청 시점의 대상 버전, 만료 시각, 승인 가능한 사용자 또는 Approval Group을 포함한다.

팀 공식 Approval Request의 원본은 중앙 Authority가 소유한다.

## Approval Decision

Approval Request에 대한 허용 또는 거부 기록. 누가 승인 또는 거부했는지, 어떤 Grant와 정책이 적용됐는지, 어떤 대상 버전에 대해 유효한지 추적한다.

## Stale Approval

승인 후 Tool arguments, 대상 파일 또는 리소스, 기준 commit, 배포 환경, 예상 비용, 위험 등급, Approval Policy, 승인 대상 버전, 실행 권한이 바뀌어 더 이상 사용할 수 없는 승인 상태.

## Policy Engine

행동 종류, 변경 대상, 변경 범위, 실행 환경, 보안 영향, 데이터 손실 가능성, 비용, 외부 시스템 영향, 프로젝트 정책, Owner Grant, Run 권한과 Tool 권한을 조합해 최종 위험도와 승인 필요 여부를 계산하는 구성 요소.

Owner 또는 Tool 구현이 위험도를 낮춰 보고할 수 없도록 최종 판단은 Policy Engine이 수행한다.

## Emergency Stop

사용자가 현재 Agent Run 취소, 새 Tool Call 차단, Worker 실행 중지 요청, Owner Grant 철회, Autonomous 모드 해제, Personal Node 연결 해제를 수행하는 긴급 제어 기능.

이미 완료된 외부 부작용은 단순 취소로 되돌아가지 않을 수 있으므로 보상 또는 복구 절차가 별도로 필요하다.

## Worker Supervisor

Owner가 만든 Task Attempt를 실행할 Worker를 준비하고 실행 상태, 로그, 실패, 아티팩트를 관리하는 로컬 실행 관리자. 사용자는 Worker를 직접 조작하지 않는다.

## Local Git Workspace

개인 또는 팀 프로젝트 작업을 수행하는 로컬 Git 작업 공간. Worker 실행 결과와 테스트는 이 작업 공간에서 생성되며, 팀 모드에서는 제출 전 Change Package의 근거가 된다.

## Work Item

사람과 Owner가 보는 목표, 기능, 버그, 조사 또는 개선 단위. 자유로운 부모·자식 트리와 kind를 가진다.

## Task

Owner가 Worker에게 주는 실행 가능한 작은 단위.

- project_id / repository_id / work_item_id
- created_by_owner_run_id
- goal
- required_capabilities
- read_scope / write_scope / forbidden_scope
- success_criteria
- dependencies
- risk_level

Worker는 할당된 Task 범위 안에서만 실행한다.

## Task Attempt

Task의 실제 한 번의 실행. 재시도할 때 기존 기록을 덮어쓰지 않는다.

- attempt_id
- task_id / repository_id
- lease_id / lease_epoch
- assigned_node_id / assigned_worker_id
- worker_run_id / worktree_id / branch_name
- model_provider / model_name / cli_adapter
- base_commit_sha / final_commit_sha / squash_commit_sha
- diff_ref / test_evidence_ref / artifact_refs
- failure_category / failure_message
- status
- started_at / completed_at

## Change Package

수정되지 않는 변경 제출 기록.

- package_id
- task_attempt_id
- origin_node_id
- base_commit / head_commit
- branch_name / changed_files
- test_evidence / artifact_refs
- referenced_decisions
- signature / supersedes_package_id

## Project Admin

프로젝트 접근, 멤버, Node 연결, 정책과 비상 조치를 관리하는 인간 사용자 역할. Project Admin은 기술적 최고 책임자가 아니며 모든 코드와 설계 변경을 직접 승인하지 않는다.

- 프로젝트 사용자 초대와 제거
- Admin, Member, Viewer 역할 관리
- 개인 서버 또는 Node 연결 승인과 해제
- 승인 정책 설정
- 프로젝트 보관과 삭제
- 비상 정지
- 계정과 Node 복구
- 다른 Admin 지정

프로젝트에는 Admin을 여러 명 둘 수 있다. 단일 Project Maintainer, 영역별 Maintainer 필수 역할, `project_maintainer_id`는 검토했지만 채택하지 않는다. 관련 결정: [[07 ADR/ADR-0004 Governance and Approval Policy]]

## Member

프로젝트에 참여하여 정책이 허용하는 범위에서 작업을 제안하고 Change Package를 제출하는 기본 사용자 역할.

## Viewer

프로젝트 상태와 허용된 산출물을 조회하는 사용자 역할.

## Approval Group

특정 코드 영역이나 기술 분야의 변경을 승인할 수 있는 사용자 그룹. 예: Frontend Approvers, Backend Approvers, Database Approvers, Security Approvers, Release Approvers.

하나의 사용자가 여러 Approval Group에 속할 수 있다. 소규모 프로젝트에서는 별도 그룹 없이 사용자 승인이나 자동 승인 정책을 사용할 수 있다.

## Approval Policy

변경 파일 경로, 영향받는 영역, Task 종류, 보안 변경, 권한 상승, 데이터 손실 가능성, 비밀정보 접근, 외부 배포, 유료 API 사용, 변경 규모, 테스트 결과 등을 기준으로 필요한 승인과 테스트를 계산하는 정책.

정책 결과는 필수 테스트, Approval Group 승인 수, 사용자 명시적 승인, Owner 자동 승인 허용 여부, Merge Queue 사용 여부 등을 포함할 수 있다. 위험 등급 R0~R4는 [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]에서 정의한다. 구체적인 정량 임계값과 승인 인원 수는 아직 확정하지 않는다.

## Decision Proposal

제품 방향과 설계 결정을 일반 코드 병합 승인과 분리해 다루는 제안 기록.

- 해결할 문제
- 고려한 대안
- Owner의 추천안
- 영향받는 영역
- 관련 기존 결정
- 예상 위험
- 되돌리기 비용
- 결정에 참여할 사용자 또는 Approval Group

## Merge Coordinator

중앙 Authority에서 Change Package를 수신하고 Approval Policy 계산, 승인 확인, 임시 통합, 필수 테스트, 충돌과 정책 위반 확인, Merge Queue 등록, 공식 브랜치 병합, 감사 이벤트 기록을 수행하는 역할.

Merge Coordinator는 정책 실행자이며 의미상 설계 충돌의 기술적 정답을 임의로 선택하지 않는다.
