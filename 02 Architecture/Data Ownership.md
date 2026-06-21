# Data Ownership

| 데이터 | 권위 있는 소유자 |
|---|---|
| 개인 프로젝트 상태 | Local Control Plane |
| Workspace, 멤버십, 프로젝트 권한 | 중앙 Authority |
| 공식 Work Item과 Task | 중앙 Authority |
| Lease와 Lock | 중앙 Authority |
| Merge Queue | 중앙 Authority |
| 공식 프로젝트 결정 | 중앙 Authority |
| Owner Conversation과 Message | Personal Node |
| Agent Run과 Agent Run Step | Personal Node |
| Tool Call | Personal Node |
| 로컬 Approval interruption 상태 | Personal Node |
| 개인 선호와 개인 메모리 | Personal Node |
| Worker 로컬 실행 기록 | Personal Node |
| 개인 Node의 중앙 프로젝트 정보 | Personal Node 캐시 |
| 코드와 Commit 이력 | Git |
| 팀 프로젝트의 공식 Change Package, Decision, Approval 결과 | 중앙 Authority |
| 승인된 Change Package와 감사 기록 | 중앙 Authority |

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]

## 원칙

- 개인 프로젝트 상태는 Local Control Plane이 소유한다.
- 팀 프로젝트의 공식 공유 상태는 중앙 Authority가 소유한다.
- Owner Conversation, Message, Agent Run, Agent Run Step, Tool Call, 로컬 Approval interruption 상태, 개인 Memory, Worker 로컬 실행 기록은 Personal Node가 소유한다.
- 팀 프로젝트의 공식 Task, Change Package, Decision과 Approval 결과는 중앙 Authority가 소유할 수 있다.
- 코드와 Commit 이력은 Git이 소유한다.
- 개인 Node의 중앙 프로젝트 정보는 캐시다.
- 개인 Node는 중앙 DB의 동등한 Writer가 아니다.

## 금지

- 개인 Node DB와 중앙 DB의 양방향 테이블 복제
- 여러 개인 Node가 같은 공유 상태의 동등한 Writer가 되는 구조
- Timestamp만으로 충돌을 덮어쓰는 Last Write Wins
