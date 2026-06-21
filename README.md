# AI Development Platform Design

이 저장소는 AI 개발 플랫폼의 제품 비전, 아키텍처, 분산 시스템 규칙, 데이터 모델, 보안 정책과 의사결정 기록을 관리한다.

## 확정된 방향

- 범용 AI 개발 플랫폼
- 개인 모드와 팀 모드는 하나의 제품과 하나의 앱으로 제공
- 사용자마다 개인 실행 환경과 개인 Owner AI 보유
- 팀 프로젝트에서는 여러 Personal Node가 중앙 Authority 서버에 연결
- 사용자는 Worker를 직접 조작하지 않고 Owner와 대화
- 중앙 서버는 공식 프로젝트 상태, 권한, 잠금, Change Package와 Merge Queue 관리
- 중앙 AI는 초기 필수 요소가 아님
- Discord 제거
- 프로젝트 결정은 대화, 권한·보안 문제는 승인 버튼
- 팀 Central Authority는 PostgreSQL, 개인 모드와 팀 Personal Node는 SQLite 사용

## 시작

1. `00 HOME.md`를 연다.
2. `10 Open Questions/Open Questions.md`의 질문에 답한다.
3. 중요한 결정은 `07 ADR/`에 기록한다.
4. 구현 작업은 별도의 코드 저장소 Issue로 만든다.
