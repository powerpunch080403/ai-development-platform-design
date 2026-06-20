# Consistency and Messaging

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

## Inbox / Outbox

Exactly-once를 가정하지 않는다. At-least-once 전달과 멱등 처리를 사용한다.

## 오프라인

가능: Owner 대화, 이미 받은 Task, 로컬 수정과 테스트.
금지: 새 공식 Lease, 새 Lock, 공식 Decision, Approval, main 병합.
