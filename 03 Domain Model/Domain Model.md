# Domain Model

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]

## App Client

사용자가 보는 하나의 앱과 UI. 개인 프로젝트에서는 Local Control Plane에 연결하고, 팀 프로젝트에서는 Personal Node와 중앙 Authority에 연결한다.

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

사용자와 개인 Owner 사이의 장기 대화 단위. 여러 Agent Run을 포함할 수 있으며 Message 기록과 Agent Run 실행 상태를 분리한다.

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

상태 후보:

- queued
- preparing
- running
- waiting_for_approval
- waiting_for_user
- waiting_for_worker
- retry_scheduled
- completed
- failed
- cancelled

구체적인 상태 전이 전체는 아직 확정하지 않는다.

## Agent Run Step

Agent Run을 구성하는 순서 있는 실행 단계. 예: context_load, model_call, tool_call, worker_dispatch, worker_result_review, approval_request, decision_record, final_response.

Run 재개 시 이미 성공한 외부 작업을 무조건 다시 실행하지 않도록 Step 결과를 기록한다.

## Tool Call

등록된 Tool을 실행하는 기록 단위. LLM은 시스템 상태나 파일을 직접 임의 변경하지 않고, 외부 부작용은 Tool Call을 통해서만 실행한다.

외부 상태를 변경하는 Tool Call에는 idempotency key를 사용한다.

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

사람과 Owner가 보는 계획 단위. 자유로운 부모·자식 관계와 kind를 가진다.

## Task

Owner가 Worker에게 주는 실행 가능한 작은 단위.

- goal
- required_capabilities
- read_scope / write_scope / forbidden_scope
- success_criteria
- dependencies
- risk_level

## Task Attempt

Task의 실제 한 번의 실행. 재시도할 때 기존 기록을 덮어쓰지 않는다.

- attempt_id
- lease_id / lease_epoch
- assigned_node_id / assigned_worker_id
- base_commit
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
