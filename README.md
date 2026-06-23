# AI Development Platform Design

이 저장소는 AI 개발 플랫폼의 제품 비전, 아키텍처, 분산 시스템 규칙, 데이터 모델, 보안 정책과 의사결정 기록을 관리한다.

## 확정된 방향

- 범용 AI 개발 플랫폼
- 현재 최우선 구현 목표는 개인 모드 MVP
- 개인 모드와 팀 모드는 하나의 제품과 하나의 앱으로 제공
- 개인 모드 MVP는 Primary Personal Server와 웹 UI 구조
- 제품 지원 목표는 Windows와 Linux, 초기 reference environment는 Ubuntu
- 사용자마다 개인 실행 환경과 개인 Owner AI 보유
- 팀 프로젝트에서는 여러 Personal Node가 중앙 Authority 서버에 연결
- 사용자는 Worker를 직접 조작하지 않고 Owner와 대화
- 개인 모드 MVP에서는 사용자 가입을 후순위로 두고 로컬 사용자 한 명을 자동 생성
- 중앙 서버는 공식 프로젝트 상태, 권한, 잠금, Change Package와 Merge Queue 관리
- 중앙 AI는 초기 필수 요소가 아님
- Discord 제거
- 프로젝트 결정은 대화, 권한·보안 문제는 승인 버튼
- 팀 Central Authority는 PostgreSQL, 개인 모드와 팀 Personal Node는 SQLite 사용

## 시작

1. `00 HOME.md`를 연다.
2. `01 Product/Personal Mode MVP.md`를 먼저 확인한다.
3. `09 Roadmap/Personal Mode MVP Roadmap.md`의 구현 순서를 확인한다.
4. `10 Open Questions/Open Questions.md`의 질문에 답한다.
5. 중요한 결정은 `07 ADR/`에 기록한다.
6. 구현 작업은 별도의 코드 저장소 Issue로 만든다.

## MVP 관련 문서

- [[01 Product/Personal Mode Complete Experience]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0015 Personal Mode Completion Scope and Design Roadmap]]
- [[07 ADR/ADR-0016 Initial Personal Mode MVP Execution Policy]]
- [[07 ADR/ADR-0017 Failure Recovery and Worktree Resume Policy]]
- [[07 ADR/ADR-0018 Personal Mode Approval Modes, Grants, and Autonomy Profiles]]
- [[09 Roadmap/Personal Mode MVP Roadmap]]
- [[11 Reviews/Initial Personal Mode MVP Design Gap Audit]]
- [[11 Reviews/Initial MVP Implementation Readiness Check]]

현재 단계는 Local Worker MVP baseline 상태이며, 외부 Worker 연동의 기본 파이프라인을 구축하는 중이다.
Real AGY Worker Alpha has been verified in controlled opt-in paths, but general user-project automation remains gated by Owner-led policy and safe pilot validation.

The design track now targets Personal Mode completion before additional implementation expansion.
현재 설계 트랙은 추가 구현 확장보다 개인 모드 완성 정의와 설계 정렬을 우선한다.
