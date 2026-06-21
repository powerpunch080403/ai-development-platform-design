# Data Ownership

| 데이터 | 권위 있는 소유자 |
|---|---|
| 개인 프로젝트 상태 | Local Control Plane |
| 개인 모드 MVP 실행 상태 | Primary Personal Server SQLite |
| ProjectRepository 메타데이터 | Primary Personal Server SQLite |
| Task와 Task Attempt 메타데이터 | Primary Personal Server SQLite |
| 로컬 사용자와 장치 연결 정보 | Primary Personal Server SQLite |
| Workspace, 멤버십, 프로젝트 권한 | 중앙 Authority |
| 공식 Work Item과 Task | 중앙 Authority |
| Lease와 Lock | 중앙 Authority |
| Merge Queue | 중앙 Authority |
| 공식 프로젝트 결정 | 중앙 Authority |
| Owner Conversation과 Message | Personal Node |
| Agent Run과 Agent Run Step | Personal Node |
| Tool Call | Personal Node |
| 로컬 Worker Lease와 Claim | Primary Personal Server SQLite |
| 로컬 Approval interruption 상태 | Personal Node |
| 개인 Owner Grant와 자율성 설정 | Personal Node |
| 개인 프로젝트의 Approval Request와 결과 | Local Control Plane 또는 Personal Node |
| 로컬 감사 기록 | Local Control Plane 또는 Personal Node |
| 개인 선호와 개인 메모리 | Personal Node |
| Worker 로컬 실행 기록 | Personal Node |
| Git Worktree 메타데이터 | Primary Personal Server SQLite |
| Commit 참조와 Merge Record | Primary Personal Server SQLite |
| Artifact 참조와 로컬 Audit Event | Primary Personal Server SQLite |
| 대형 로그, 아티팩트, 테스트 출력 | 파일 저장소 |
| Artifact 파일 본문 | Artifact Store |
| UI Client 브라우저 상태 | 원본 아님 |
| 개인 Node의 중앙 프로젝트 정보 | Personal Node 캐시 |
| 코드, Commit, Branch와 Merge 이력 | Git |
| 팀 프로젝트 Approval Policy와 Approval Group | 중앙 Authority |
| 팀 공식 Approval Request와 결과 | 중앙 Authority |
| 중앙 Merge Queue와 공식 Merge Record | 중앙 Authority PostgreSQL |
| 공식 Grant 또는 중앙 위임 | 중앙 Authority |
| 팀 프로젝트의 공식 Change Package, Decision, Approval 결과 | 중앙 Authority |
| 팀 감사 기록 | 중앙 Authority |
| 승인된 Change Package와 감사 기록 | 중앙 Authority |

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]

## 원칙

- 개인 프로젝트 상태는 Local Control Plane이 소유한다.
- 개인 모드 MVP의 공식 실행 상태, 사용자와 장치 연결 정보, ProjectRepository, Task Attempt, Worktree, Commit 참조, Merge Record, Artifact 참조와 Audit Event 메타데이터는 Primary Personal Server의 SQLite가 원본이다.
- 대형 로그, 바이너리 아티팩트와 테스트 출력은 파일 저장소가 원본일 수 있으며 SQLite에는 참조를 저장할 수 있다.
- 브라우저는 공식 상태의 원본이 아니다.
- 운영체제는 데이터 소유권에 영향을 주지 않는다.
- 팀 프로젝트의 공식 공유 상태는 중앙 Authority가 소유한다.
- Owner Conversation, Message, Agent Run, Agent Run Step, Tool Call, 로컬 Approval interruption 상태, 개인 Memory, Worker Run과 Worker Lease/Claim 기록은 Personal Node SQLite가 소유한다.
- 큰 로그와 산출물 파일은 Artifact Store가 소유하고 SQLite는 `artifact_ref`를 소유한다.
- 개인 Owner Grant, 개인 자율성 설정, 개인 프로젝트의 Approval Request와 결과, 로컬 감사 기록은 Local Control Plane 또는 Personal Node가 소유한다.
- 팀 프로젝트의 공식 Task, Change Package, Decision과 Approval 결과는 중앙 Authority가 소유할 수 있다.
- 팀 프로젝트 Approval Policy, Approval Group, 팀 공식 Approval Request와 결과, 공식 Grant 또는 중앙 위임, 팀 감사 기록은 중앙 Authority가 소유한다.
- 팀 공식 상태 변경, 중앙 Merge Queue와 공식 merge 기록은 Central Authority PostgreSQL이 소유한다.
- 로컬 기록과 중앙 기록은 `correlation_id`, `central_request_id`, `agent_run_id`, `task_attempt_id` 또는 `change_package_id` 같은 연결 식별자로 추적한다.
- Personal Node의 팀 승인 데이터는 캐시 또는 실행 재개 상태이며 공식 원본이 아니다.
- 코드, Commit, Branch와 Merge 이력은 Git이 소유한다. SQLite의 Git 관련 엔티티는 실행 복구와 감사에 필요한 참조이며 Git 객체와 이력의 원본을 대체하지 않는다.
- 개인 Node의 중앙 프로젝트 정보는 캐시다.
- 개인 Node는 중앙 DB의 동등한 Writer가 아니다.

## 금지

- 개인 Node DB와 중앙 DB의 양방향 테이블 복제
- 여러 개인 Node가 같은 공유 상태의 동등한 Writer가 되는 구조
- Timestamp만으로 충돌을 덮어쓰는 Last Write Wins
