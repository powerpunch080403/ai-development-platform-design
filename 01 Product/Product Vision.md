---
type: product
status: draft
updated: 2026-06-21
---

# Product Vision

## 비전

개발자는 하나의 앱에서 개인 모드와 팀 모드를 오가며 자기 Owner AI와 대화해 프로젝트를 계획하고, Owner는 로컬 Worker에게 실행 가능한 작업을 배정한다.

개인 모드와 팀 모드는 별개의 제품이 아니다. 하나의 앱, 하나의 UI, 공통 도메인 모델, 공통 Owner 및 Worker 개념을 사용한다. 개인 모드는 팀 모드 공통 핵심에서 팀 전용 기능을 제거하거나 숨긴 축소형이다.

개인 프로젝트에서는 App UI가 Local Control Plane에 연결된다. 팀 프로젝트에서는 각 사용자의 Personal Node가 중앙 Authority 서버에 연결된다. 중앙 Authority는 공식 프로젝트 상태, 권한, 작업 소유권, 변경 패키지, 잠금과 Merge Queue를 관리한다.

관련 결정:

- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]

## 사용자 경험

- 사용자는 Worker를 직접 조작하지 않는다.
- 사용자는 개인 Owner와 대화한다.
- 개인 Owner는 사용자별 실행 서버에서 동작한다.
- Owner는 단일 영구 AI 세션이 아니라 Owner Supervisor와 복구 가능한 Agent Run들로 구성된다.
- Owner가 계획, 작업 분해, Worker 선택, 결과 검토와 재시도를 담당한다.
- 프로젝트 방향은 대화로 결정한다.
- 개인 서버 또는 연결된 장치 연결, 비밀 접근, 권한 상승, 위험 명령과 병합은 승인 UI로 처리한다.
- 기본값은 사용자 승인과 최소 권한이다.
- 사용자는 범위가 제한된 Owner 자율 권한을 선택할 수 있다.
- 팀 정책과 다른 사용자의 권한은 개인 Grant로 우회할 수 없다.
- 고위험 작업은 위험 기반 승인 UI에서 대상, 범위, 위험, 되돌리기 가능성을 확인한다.

## 모드

개인 모드:

- 별도의 중앙 Authority 서버 없이 Local Control Plane이 개인 프로젝트 상태를 관리한다.
- 사용자는 자동으로 Admin이다.
- 팀원 초대, Approval Group, 여러 사용자 간 권한 관리, 중앙 Merge Queue는 숨기거나 비활성화한다.

팀 모드:

- 각 사용자가 자신의 Personal Node를 가진다.
- 공식 공유 상태는 중앙 Authority가 관리한다.
- 개인 Node는 중앙 DB의 동등한 Writer가 아니다.

## Non-goals

- Discord 운영 UI
- 초기 멀티 마스터 DB
- 초기 중앙 의사결정 AI
- 초기 Enterprise SSO와 다중 조직 SaaS
- 초기 마이크로서비스 분리
