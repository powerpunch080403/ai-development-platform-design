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

관련 문서:

- [[07 ADR/ADR-0004 Governance and Approval Policy]]
- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]

## 우선 답할 질문

1. Node 등록과 인증 방식은 무엇인가?
2. 구체적인 Agent Run 상태 전이는 무엇인가?
3. 재시도와 timeout 값은 무엇인가?
4. 모델 선택 및 fallback 정책은 무엇인가?
5. Context 압축 및 검색 방식은 무엇인가?
6. SSE와 WebSocket 중 무엇을 사용할 것인가?
7. Durable Workflow 엔진 도입 시점은 언제인가?
8. Run 동시성 제한은 무엇인가?
9. Owner 자동 병합과 완전 자동 실행의 허용 범위는 어디까지인가?
10. Approval Policy의 위험도 단계, 승인 인원 수, 승격 규칙은 어떻게 정의할 것인가?
11. 데스크톱 앱 포장 방식과 프론트엔드 세부 기술은 무엇인가?
12. 새 v2 저장소의 실제 생성 및 마이그레이션 방식은 무엇인가?
