# Database Strategy

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

## 개인 모드

SQLite + SQLAlchemy 2 + Alembic을 사용한다. 첫 구현 Slice에서 migration baseline을 만들며 정확한 DDL과 index는 skeleton 구현 과정에서 현재 ADR의 무결성 규칙에 맞게 작성한다.

저장 데이터 후보:

- local_users
- account_links
- devices
- sessions
- pairing_codes
- projects
- project_repositories
- project_settings
- conversations
- messages
- agent_runs
- agent_run_steps
- tool_definitions
- tool_calls
- work_items
- tasks
- task_attempts
- worker_runs
- worker_claims
- worker_leases
- test_runner_nodes
- test_runs
- worker_reports
- run_checkpoints
- owner_memories
- pending_approvals
- approval_interruptions
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
- branch_refs
- commit_refs
- merge_records
- artifact_refs
- local_change_packages
- app_settings
- audit_events

개인 모드에서는 Local Control Plane이 한 사용자 중심의 로컬 데이터 저장소를 관리한다.

Runtime SQLite DB는 App-managed Local Runtime의 app data directory에 두고 Public source repository에 commit하지 않는다. Alembic migration source만 구현 repository에서 version control한다.

개인 프로젝트의 Approval Request와 결과, 개인 Owner Grant, 개인 자율성 설정, 로컬 감사 기록은 SQLite가 공식 원본이다.

개인 모드 MVP의 SQLite 데이터 경로는 운영체제별 기본 데이터 경로 Resolver와 설정으로 정한다. `/home/...`, `/var/...`, `C:\Users\...`, `C:\ProgramData\...` 같은 경로를 코드에 고정하지 않는다.

ProjectRepository, Work Item, Task, Task Attempt, Conversation, Agent Run과 Git 메타데이터의 관계, 상태 전이와 무결성 규칙은 [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]을 따른다. 구체적인 전체 스키마, 타입, 인덱스와 DDL은 후속 데이터 모델 설계로 남긴다.

`tool_calls`는 [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]의 공통 Envelope에 따라 Tool 이름과 버전, caller, 관련 도메인 ID, risk level, idempotency key, correlation ID, arguments, 상태, 결과·오류 참조와 시간을 추적한다. Approval Interruption은 Approval Request와 Agent Run의 대기 상태를 연결한다. 큰 Tool 결과와 실행 증거는 `artifact_refs`로 Artifact Store를 참조한다.

상태 전이와 외부 부작용은 `runtime_events`와 `audit_events`에 기록한다. Tool Registry의 정확한 저장 방식, 전체 schema, index, lease timeout과 artifact 보존 정책은 후속 설계로 남긴다.

`account_links`는 nullable 중앙 `account_id`와 link status를 local user에 연결한다. `devices`는 device type, name과 `last_seen_at`을, `sessions`는 `device_id`, `local_user_id`, nullable `account_id`, `token_hash`, `last_seen_at`, `idle_expires_at`, `absolute_expires_at`, `revoked_at`을 추적한다. `pairing_codes`에는 code 원문이 아니라 hash, 만료와 사용 시각을 저장한다.

중앙 account 원본과 다중 기기 동기화는 Personal Mode MVP SQLite의 책임이 아니다. 동기화 schema, conflict resolution과 중앙 Session 모델은 후속 설계로 남긴다.

`test_runner_nodes`는 Node status, capabilities, network kind, 허용 Project/Repository, `last_seen_at`과 폐기를 추적한다. `test_runs`는 Task Attempt, Worker Run, Runner Node, repository, `commit_sha`, environment, test target, command summary, status, timeout과 `retry_of_test_run_id`를 연결한다. `worker_reports`는 Test Run 분석, failure hypothesis, retry 변경과 `artifact_refs`를 기록한다.

Test Run artifact 본문은 Artifact Store에 저장하고 `artifact_refs`가 `test_run_id`와 연결된다. 정확한 heartbeat 주기, network protocol, command policy와 artifact retention은 후속 설계다.

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
