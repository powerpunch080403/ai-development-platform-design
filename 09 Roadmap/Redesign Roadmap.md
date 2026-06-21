# Redesign Roadmap

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]

## Phase 0: 설계 저장소

- 비전, 용어, 데이터 소유권 확정
- 핵심 ADR 작성
- 기존 레포 감사
- Open Questions 정리

## Phase 1: 공통 도메인과 개인 모드

- 공통 도메인 모델
- 개인 모드 프로젝트 흐름
- 개인 Owner 대화
- Task / Attempt

## Phase 2: Local Control Plane

- 개인 프로젝트 상태 관리
- Work Item과 Task 관리
- Task Attempt 관리
- 로컬 승인과 로컬 병합
- 실패 복구

## Phase 3: Owner Runtime과 Worker Supervisor

- Conversation과 Message
- Owner Supervisor
- Agent Run 상태 머신
- Tool Registry와 Tool Call
- Approval interruption
- Worker dispatch와 결과 재개
- Context와 Memory
- UI 이벤트 스트리밍
- 실패 복구
- Worker Supervisor와 로컬 Worker 실행 로그

## Phase 4: 개인 모드 SQLite

- owner_conversations, owner_messages
- agent_runs, agent_run_steps, tool_calls
- run_checkpoints, pending_approvals, runtime_events
- owner_memories
- Work Item, Task, Task Attempt
- Worker 실행 기록
- 로컬 Change Package와 설정

## Phase 5: 단일 앱 UI

- 개인 프로젝트는 Local Control Plane에 연결
- 팀 프로젝트는 Personal Node와 중앙 Authority에 연결
- 팀 전용 기능 숨김과 비활성화
- 승인 UI

## Phase 6: 중앙 Authority와 PostgreSQL

- PostgreSQL
- 프로젝트 멤버십
- Lease와 Scope Lock
- Change Package와 Merge Queue
- Approval Policy
- Audit Event

## Phase 7: Personal Node 연결

- Node 등록
- Inbox / Outbox
- 중앙 상태 캐시
- 제출 전 Change Package
- 오프라인 작업 상태

## Phase 8: Team Federation

- 여러 개인 Node 연결
- 오프라인/재연결
- 의미상 충돌 전달
- 영역별 권한자

## 연기

- Enterprise SSO
- 다중 조직 SaaS
- 중앙 Coordination AI
- 고가용성
- 대형 메시지 브로커
