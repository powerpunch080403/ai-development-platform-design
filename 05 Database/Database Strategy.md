# Database Strategy

## 중앙 Authority

PostgreSQL 우선.

- 여러 개인 Node의 동시 요청
- Row Lock과 트랜잭션
- 제약조건
- 안정적인 마이그레이션과 백업

후보 테이블:

- workspaces, users, workspace_memberships
- nodes, node_credentials
- projects, project_memberships
- work_items, tasks, task_attempts, task_dependencies
- leases, scope_locks
- change_packages, test_evidence
- merge_queue_entries
- decisions
- approval_requests, approval_decisions
- audit_events, outbox_events, processed_commands

## 개인 Node

SQLite 우선.

- owner_conversations, owner_messages, owner_memories
- local_workers, local_worker_runs
- pending_commands, inbox_events
- cached_projects, cached_tasks
- local_change_packages

## 금지

- 중앙 SQLite를 네트워크 파일로 공유
- 개인 Node가 중앙 DB에 직접 접속
- 중앙/로컬 DB의 무분별한 양방향 복제
