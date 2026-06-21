# Database Strategy

관련 결정: [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]

## 개인 모드

SQLite를 사용한다.

저장 데이터:

- Owner 대화
- 개인 메모리
- Work Item
- Task
- Task Attempt
- Worker 실행
- 승인
- 로컬 Change Package
- 설정

개인 모드에서는 Local Control Plane이 한 사용자 중심의 로컬 데이터 저장소를 관리한다.

## 팀 Personal Node

SQLite를 사용한다.

저장 데이터:

- Owner 대화
- 개인 메모리
- 로컬 Worker 실행
- Inbox와 Outbox
- 오프라인 작업
- 중앙 상태 캐시
- 제출 전 Change Package

Personal Node의 SQLite는 중앙 프로젝트 정보 일부를 캐시할 수 있지만, 중앙 Authority DB의 복제본이나 동등한 Writer가 아니다.

## 팀 Central Authority

PostgreSQL을 사용한다.

저장 데이터:

- 공유 프로젝트 상태의 유일한 원본
- 멤버십
- Task Lease
- Scope Lock
- Change Package
- Approval Policy
- Merge Queue
- Audit Event

## 금지

- SQLite 파일을 중앙 DB처럼 네트워크로 공유
- 개인 Node가 중앙 DB에 직접 접속
- 개인 SQLite와 중앙 PostgreSQL의 양방향 테이블 복제
