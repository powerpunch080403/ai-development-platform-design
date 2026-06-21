# Database Strategy

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]

## 개인 모드

SQLite를 사용한다.

저장 데이터 후보:

- local_users
- connected_devices
- device_sessions
- projects
- project_settings
- owner_conversations
- owner_messages
- agent_runs
- agent_run_steps
- tool_calls
- work_items
- tasks
- task_attempts
- worker_runs
- run_checkpoints
- owner_memories
- pending_approvals
- autonomy_profiles
- owner_grants
- approval_requests
- approval_decisions
- policy_evaluations
- grant_revocations
- security_audit_events
- budget_usage
- runtime_events
- git_worktrees
- artifact_refs
- local_change_packages
- app_settings

개인 모드에서는 Local Control Plane이 한 사용자 중심의 로컬 데이터 저장소를 관리한다.

개인 프로젝트의 Approval Request와 결과, 개인 Owner Grant, 개인 자율성 설정, 로컬 감사 기록은 SQLite가 공식 원본이다.

개인 모드 MVP의 SQLite 데이터 경로는 운영체제별 기본 데이터 경로 Resolver와 설정으로 정한다. `/home/...`, `/var/...`, `C:\Users\...`, `C:\ProgramData\...` 같은 경로를 코드에 고정하지 않는다.

구체적인 전체 스키마와 인덱스는 후속 데이터 모델 설계로 남긴다.

## 팀 Personal Node

SQLite를 사용한다.

저장 데이터:

- Owner 대화
- 개인 메모리
- owner_conversations
- owner_messages
- agent_runs
- agent_run_steps
- tool_calls
- run_checkpoints
- owner_memories
- pending_approvals
- autonomy_profiles
- owner_grants
- approval_requests
- approval_decisions
- policy_evaluations
- grant_revocations
- security_audit_events
- budget_usage
- runtime_events
- 로컬 Worker 실행
- Inbox와 Outbox
- 오프라인 작업
- 중앙 상태 캐시
- 제출 전 Change Package

Personal Node의 SQLite는 중앙 프로젝트 정보 일부를 캐시할 수 있지만, 중앙 Authority DB의 복제본이나 동등한 Writer가 아니다.

Owner Runtime 관련 테이블의 구체적인 최종 스키마, 인덱스, 보존 정책, 상태 전이 제약은 후속 설계로 남긴다.

팀 프로젝트의 승인 데이터가 Personal Node SQLite에 있을 때는 표시, 오프라인 작업, Agent Run 재개를 위한 캐시 또는 대기 상태다. 팀 공식 승인 원본은 아니다.

## 팀 Central Authority

PostgreSQL을 사용한다.

저장 데이터:

- 공유 프로젝트 상태의 유일한 원본
- 멤버십
- Task Lease
- Scope Lock
- Change Package
- Approval Policy
- Approval Group
- Approval Request와 Approval Decision
- 공식 Grant 또는 중앙 위임
- Policy Evaluation
- Grant Revocation
- Security Audit Event
- Budget Usage
- Merge Queue
- Audit Event

팀 프로젝트 Approval Policy, Approval Group, 팀 공식 Approval Request와 결과, 공식 Grant 또는 중앙 위임, 팀 감사 기록은 PostgreSQL이 공식 원본이다.

구체적인 최종 스키마는 후속 설계로 남긴다.

## 금지

- SQLite 파일을 중앙 DB처럼 네트워크로 공유
- 개인 Node가 중앙 DB에 직접 접속
- 개인 SQLite와 중앙 PostgreSQL의 양방향 테이블 복제
