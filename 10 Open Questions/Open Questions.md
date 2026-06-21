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
- 첫 UI는 설치형 데스크톱 앱이 아니라 브라우저 웹 UI다.
- 개인 모드 MVP의 원격 접속은 Tailscale을 기본 전제로 한다.
- 기존 서버 Git 저장소 가져오기부터 구현한다.
- Model Provider Adapter는 API와 CLI를 지원할 수 있게 설계하고 MVP는 CLI 우선이다.
- 첫 Worker는 Generic Development Worker 하나다.
- Worker는 격리된 Git Worktree 안에서 기본 자동 작업할 수 있다.
- 기본 브랜치 병합 전에는 사용자 승인이 필요하다.
- 프로젝트별 자율성 설정이 가능하다.

관련 문서:

- [[07 ADR/ADR-0004 Governance and Approval Policy]]
- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]

## 우선 답할 질문

1. Node 등록과 인증 방식은 무엇인가?
2. 구체적인 Agent Run 상태 전이는 무엇인가?
3. 재시도와 timeout 값은 무엇인가?
4. 모델 선택 및 fallback 정책은 무엇인가?
5. Context 압축 및 검색 방식은 무엇인가?
6. SSE와 WebSocket 중 무엇을 사용할 것인가?
7. Durable Workflow 엔진 도입 시점은 언제인가?
8. Run 동시성 제한은 무엇인가?
9. 각 위험 등급의 정량 임계값은 무엇인가?
10. 비용 예산의 단위와 기본값은 무엇인가?
11. 재인증이 필요한 Action 목록은 무엇인가?
12. 팀 다중 승인 수와 승격 규칙은 어떻게 정의할 것인가?
13. 보호 영역을 정의하는 UI와 파일 형식은 무엇인가?
14. Emergency Stop의 프로세스 종료 범위는 어디까지인가?
15. 권한 철회와 이미 실행 중인 Tool Call의 경합은 어떻게 처리할 것인가?
16. Linux systemd 서비스 설치 방식은 어떻게 정의할 것인가?
17. Windows Service와 사용자 세션 기반 백그라운드 실행 중 무엇을 우선할 것인가?
18. 운영체제별 기본 데이터 경로 Resolver는 어떻게 정의할 것인가?
19. macOS 공식 지원 시점은 언제인가?
20. 첫 번째 실제 CLI Adapter 종류는 무엇인가?
21. CLI별 구조화된 출력 방식은 무엇인가?
22. 장치 토큰 형식과 만료 정책은 무엇인가?
23. 허용 프로젝트 루트의 운영체제별 기본 경로는 무엇인가?
24. 브랜치와 Worktree 명명 규칙은 무엇인가?
25. Git commit을 기본 자동 허용할지 후보 상태로만 둘지?
26. 병합 방식은 merge, squash, rebase 중 무엇인가?
27. SQLite 백업 방식은 무엇인가?
28. 로그와 아티팩트 보존 기간은 무엇인가?
29. Emergency Stop의 정확한 종료 범위는 어디까지인가?
30. 추가 Worker Host를 도입할 시점은 언제인가?
31. 자동 업데이트 방식은 무엇인가?
32. 데스크톱 앱 포장 방식과 프론트엔드 세부 기술은 무엇인가?
33. 새 v2 저장소의 실제 생성 및 마이그레이션 방식은 무엇인가?
