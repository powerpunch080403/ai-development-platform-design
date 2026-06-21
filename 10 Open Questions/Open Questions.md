# Open Questions

## Resolved Decisions

- 단일 Project Maintainer는 사용하지 않는다.
- 영역별 Maintainer 필수 역할은 사용하지 않는다.
- 기본 역할은 Admin, Member, Viewer이다.
- 기술 승인은 Approval Group과 Approval Policy로 관리한다.
- 실제 병합은 Merge Coordinator가 수행한다.
- 의미상 충돌은 Decision Proposal과 정책 기반 절차로 처리한다.
- Project Admin은 모든 기술 결정을 직접 내리는 역할이 아니다.
- 개인 모드에서 Worker 결과의 최종 병합은 기본적으로 사용자 승인이고, Owner 자동 병합은 선택 설정이다.
- 개인 모드와 팀 모드는 별개의 제품이 아니라 하나의 앱과 하나의 UI로 제공한다.
- 개인 모드는 팀 모드 공통 핵심에서 팀 전용 기능을 제거하거나 숨긴 형태다.
- 개인 모드에서는 중앙 Authority 없이 Local Control Plane, Owner Runtime, Worker Supervisor, SQLite, Local Git Workspace로 개인 프로젝트 작업을 수행한다.
- 개인 프로젝트 상태는 Local Control Plane이 소유한다.
- 팀 프로젝트의 공식 공유 상태는 중앙 Authority가 소유한다.
- 개인 모드와 팀 Personal Node는 SQLite를 사용한다.
- 팀 Central Authority는 PostgreSQL을 사용한다.
- 팀 모드의 PostgreSQL은 중앙 Authority 도입 단계에서 사용한다.
- Owner는 사용자별 Personal Node에서 실행한다.
- 중앙 Authority에는 초기 버전에서 필수 중앙 AI를 두지 않는다.
- Owner Supervisor는 장기 실행 서비스다.
- 실제 AI 작업은 요청별 Agent Run으로 실행한다.
- Agent Run 상태는 저장 및 재개 가능하다.
- Conversation과 Agent Run은 분리한다.
- Run State와 장기 Memory는 분리한다.
- Owner 자동 실행의 기본값은 사용자 승인과 최소 권한이다.
- 범위가 제한된 Grant를 통해 R0~R3 작업 자동 실행이 가능하다.
- Autonomous 프로필을 제공한다.
- R4 작업은 자동 실행할 수 없다.
- 팀 정책과 다른 사용자의 권한은 개인 Grant로 우회할 수 없다.
- Owner는 자신의 권한을 확대할 수 없다.
- 개인 모드 MVP는 단일 Ubuntu 배포 대상으로 한정하지 않는다.
- Primary Personal Server Runtime은 Windows와 Linux 지원을 목표로 한다.
- Ubuntu Linux는 초기 구현과 검증을 위한 Ubuntu reference environment다.
- macOS는 첫 MVP의 공식 지원 범위에 포함하지 않는다.
- 개인 모드 MVP에서는 사용자 가입을 구현하지 않고 첫 설치 시 로컬 사용자 한 명을 자동 생성한다.
- user 엔티티, user_id 참조, 장치와 사용자 관계, 프로젝트 소유권, 세션과 권한 구조는 유지한다.
- 첫 구현 목표는 팀 모드가 아니라 개인 모드 MVP다.
- 개인 모드 MVP는 Primary Personal Server 중심으로 배치한다.
- 제품의 공식 장기 UI 목표는 Desktop App이며, 초기 MVP는 Desktop App Shell에 재사용 가능한 browser-accessible Web UI로 시작한다.
- 개인 모드 MVP의 원격 접속은 Tailscale을 기본 전제로 한다.
- 기존 서버 Git 저장소 가져오기부터 구현한다.
- Model Provider Adapter는 API와 CLI를 지원할 수 있게 설계하고 MVP는 CLI 우선이다.
- 첫 Worker는 Generic Development Worker 하나다.
- Worker는 격리된 Git Worktree 안에서 기본 자동 작업할 수 있다.
- 기본 `Allow Local Work`에서는 기본 브랜치 병합 전에 사용자 승인이 필요하다.
- 프로젝트별 자율성 설정이 가능하다.
- Project는 하나 이상의 ProjectRepository를 가지며 여러 Git Repository를 포함할 수 있다.
- Personal Mode MVP의 첫 작업 실행은 primary repository 중심으로 시작한다.
- Work Item은 자유로운 부모·자식 트리를 가진다.
- Task는 Owner가 만들고 Worker가 할당된 범위에서 실행하며, Worker는 Task 없이 실행하지 않는다.
- Conversation과 Agent Run은 분리하고 일반 대화 중 실행 요청은 새 Agent Run과 필요한 Task로 이어진다.
- Owner는 명시적인 Tool Call을 통해서만 SQLite, Git, 파일시스템과 Worker 상태를 변경한다.
- Worker는 Task Attempt별 작업 브랜치에 결과를 자동 commit한다.
- Personal Mode MVP의 기본 브랜치 반영 방식은 squash merge다.
- dirty repository에서는 등록과 읽기·분석은 허용하지만 해당 repository의 Worker 작업 시작은 차단한다.
- Worker는 Owner 검토와 승인 절차 없이 기본 브랜치에 병합할 수 없다.
- 하나의 Task에는 여러 Task Attempt가 있을 수 있고 재시도는 기존 Attempt를 덮어쓰지 않는다.
- HTTP API는 브라우저 전용이 아니라 UI Shell을 서버 기능에 연결하는 첫 local transport다.
- Internal Tool Contract를 먼저 정의하고 HTTP API는 동일 application service를 호출하는 transport로 둔다.
- Tool은 일반 AI 능력 확장이 아니라 플랫폼이 소유한 서버 상태와 부작용을 통제하는 기능이다.
- 최종 Tool 목록은 고정하지 않고 Tool Registry에서 버전, schema와 정책 메타데이터를 관리한다.
- 모든 Tool Call은 공통 Envelope를 가지며 부작용 Tool에는 idempotency key가 필요하다.
- Tool Boundary와 Policy Engine은 각각 정상 실행 틀과 호출 권한·승인 여부를 판단한다.
- Personal Mode는 Primary Personal Server 내부의 Local Policy Engine을 사용한다.
- 승인 필요 시 Approval Interruption을 만들고 Agent Run을 `waiting_for_approval`로 전환한다.
- Worker lease/claim과 `artifact_refs`는 Personal Mode MVP에 포함한다.
- Command Policy는 사용자나 조직이 실행 환경 경계를 표현하는 설정이다.
- Personal Mode의 squash merge 승인은 Owner Grant와 Autonomy Profile로 평가하며 기본 `Allow Local Work`에서는 사용자 승인이 필요하다.

관련 문서:

- [[07 ADR/ADR-0004 Governance and Approval Policy]]
- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]

## 우선 답할 질문

1. Node 등록과 인증 방식은 무엇인가?
2. Agent Run과 Task 재시도의 횟수, backoff와 timeout 값은 무엇인가?
3. 모델 선택 및 fallback 정책은 무엇인가?
4. Context 압축 및 검색 방식은 무엇인가?
5. SSE와 WebSocket 중 무엇을 사용할 것인가?
6. Durable Workflow 엔진 도입 시점은 언제인가?
7. Run 동시성 제한은 무엇인가?
8. 각 위험 등급의 정량 임계값은 무엇인가?
9. 비용 예산의 단위와 기본값은 무엇인가?
10. 재인증이 필요한 Action 목록은 무엇인가?
11. 팀 다중 승인 수와 승격 규칙은 어떻게 정의할 것인가?
12. 보호 영역을 정의하는 UI와 파일 형식은 무엇인가?
13. Emergency Stop의 프로세스 종료 범위는 어디까지인가?
14. 권한 철회와 이미 실행 중인 Tool Call의 경합은 어떻게 처리할 것인가?
15. Linux systemd 서비스 설치 방식은 어떻게 정의할 것인가?
16. Windows Service와 사용자 세션 기반 백그라운드 실행 중 무엇을 우선할 것인가?
17. 운영체제별 기본 데이터 경로 Resolver는 어떻게 정의할 것인가?
18. macOS 공식 지원 시점은 언제인가?
19. 첫 번째 실제 CLI Adapter 종류는 무엇인가?
20. CLI별 구조화된 출력 방식은 무엇인가?
21. 장치 토큰 형식과 만료 정책은 무엇인가?
22. 허용 프로젝트 루트의 운영체제별 기본 경로는 무엇인가?
23. 브랜치와 Worktree 명명 규칙은 무엇인가?
24. SQLite 백업 방식은 무엇인가?
25. artifact 보존 기간과 용량 정책은 무엇인가?
26. 추가 Worker Host를 도입할 시점은 언제인가?
27. 자동 업데이트 방식은 무엇인가?
28. Desktop App Shell은 Tauri, Electron 또는 다른 기술 중 무엇을 사용하는가?
29. 새 v2 저장소의 실제 생성 및 마이그레이션 방식은 무엇인가?
30. multi-repository 작업의 원자적 병합 정책은 무엇인가?
31. 한 repository의 dirty 상태가 다른 clean repository 작업을 차단해야 하는가?
32. 자동 commit 비활성화 옵션의 UX는 무엇인가?
33. merge conflict의 재시도 UX는 무엇인가?
34. 정확한 SQLite 인덱스와 DDL은 무엇인가?
35. cross-repository Change Package는 어떻게 설계하는가?
36. 팀 모드 Approval Group과 Merge Coordinator의 세부 상태 머신은 무엇인가?
37. local IPC를 사용할 것인가? 사용한다면 어떤 방식을 선택하는가?
38. OS keychain은 어떤 방식으로 연동하는가?
39. HTTP API 인증, 세션과 토큰 만료 정책은 무엇인가?
40. 최종 Tool 목록과 각 Tool의 input/output schema는 무엇인가?
41. Tool version 호환성과 migration은 어떻게 처리하는가?
42. Worker lease timeout, heartbeat와 reclaim 규칙은 무엇인가?
43. Worker package/report의 상세 형식은 무엇인가?
44. artifact compression과 deletion 정책은 무엇인가?
45. 팀 모드 Memory 구조는 무엇인가?
46. Enterprise command policy의 상세 규칙은 무엇인가?
47. Golden Path의 최종 end-to-end 테스트 시나리오는 무엇인가?
48. Central Authority와 Personal Node 사이의 정책 캐시·위임 모델은 무엇인가?
