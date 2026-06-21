# Consistency and Messaging

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]

## 일관성

중앙 Authority가 공유 프로젝트 상태의 단일 Writer다.

강한 일관성이 필요한 상태:

- Task 소유권
- Lease와 Scope Lock
- 권한 변경과 Approval
- Merge Queue
- 공식 Decision

최종적 일관성을 허용할 상태:

- 진행률
- Worker heartbeat
- 알림
- 검색 인덱스
- 통계와 개인 Node 캐시

## Lease

- lease_id: UUID
- lease_epoch: 증가하는 세대 번호
- leased_node_id
- expires_at

모든 heartbeat와 결과 제출에 lease_id와 lease_epoch를 넣는다.

## Idempotency

모든 변경 명령에 command_id를 넣고 중복 요청에는 기존 결과를 반환한다.

Tool Call도 멱등성을 고려한다. 외부 상태를 변경하는 Tool Call에는 idempotency key를 사용하고, Run 재개 시 이미 성공한 Step의 외부 부작용을 중복 실행하지 않는다.

## Inbox / Outbox

Exactly-once를 가정하지 않는다. At-least-once 전달과 멱등 처리를 사용한다.

## Agent Run 복구

Agent Run 상태는 지속 저장한다. 승인과 Worker 결과는 Run을 재개시키는 외부 이벤트다.

- 승인 대기 상태는 실패가 아니라 `waiting_for_approval` 같은 재개 가능한 상태다.
- Worker 결과 대기 상태는 실패가 아니라 `waiting_for_worker` 같은 재개 가능한 상태다.
- 이벤트 스트림이 끊겨도 DB의 공식 Run 상태가 원본이다.
- 같은 프로젝트의 변경 명령은 Project Command Queue 같은 순서 제어가 필요하다.
- 같은 Task나 Change Package를 변경하는 명령에는 version 또는 idempotency 검사를 적용한다.

## 오프라인

가능: Owner 대화, 이미 받은 Task, 로컬 수정과 테스트.
금지: 새 공식 Lease, 새 Lock, 공식 Decision, Approval, main 병합.
