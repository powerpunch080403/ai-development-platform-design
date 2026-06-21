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

관련 문서: [[07 ADR/ADR-0004 Governance and Approval Policy]]

## 우선 답할 질문

1. 개인 모드에서 Authority 없이 어느 범위까지 제공할 것인가?
2. 개인 Node UI와 중앙 Authority UI를 하나의 앱으로 제공할 것인가?
3. Node 등록에 공개키, mTLS, 장치 코드 중 무엇을 사용할 것인가?
4. Owner Runtime은 장기 세션인가, 요청별 Agent Run인가?
5. 기존 레포를 점진적으로 마이그레이션할지 v2를 새로 구성할지?
6. 중앙 PostgreSQL 전환 시점은 언제인가?
7. Frontend 프레임워크는 무엇으로 할 것인가?
8. Approval Policy의 위험도 단계, 승인 인원 수, 승격 규칙은 어떻게 정의할 것인가?
9. Owner 자동 병합과 완전 자동 실행의 허용 범위는 어디까지인가?
