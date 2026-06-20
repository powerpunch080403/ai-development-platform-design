---
type: product
status: draft
updated: 2026-06-20
---

# Product Vision

## 비전

개발자는 자기 개인 서버의 Owner AI와 대화해 프로젝트를 계획하고, Owner는 로컬 Worker에게 실행 가능한 작업을 배정한다.

팀 프로젝트에서는 각 사용자의 개인 서버가 중앙 Authority 서버에 연결된다. 중앙 서버는 공식 프로젝트 상태, 권한, 작업 소유권, 변경 패키지, 잠금과 Merge Queue를 관리한다.

## 사용자 경험

- 사용자는 Worker를 직접 조작하지 않는다.
- 사용자는 개인 Owner와 대화한다.
- Owner가 계획, 작업 분해, Worker 선택, 결과 검토와 재시도를 담당한다.
- 프로젝트 방향은 대화로 결정한다.
- Node 연결, 비밀 접근, 권한 상승, 위험 명령과 병합은 승인 UI로 처리한다.

## Non-goals

- Discord 운영 UI
- 초기 멀티 마스터 DB
- 초기 중앙 의사결정 AI
- 초기 Enterprise SSO와 다중 조직 SaaS
- 초기 마이크로서비스 분리
