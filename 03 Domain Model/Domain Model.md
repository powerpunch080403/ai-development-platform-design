# Domain Model

관련 결정: [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]

## App Client

사용자가 보는 하나의 앱과 UI. 개인 프로젝트에서는 Local Control Plane에 연결하고, 팀 프로젝트에서는 Personal Node와 중앙 Authority에 연결한다.

## Local Control Plane

개인 모드에서 개인 프로젝트 상태를 관리하는 로컬 제어 계층.

- Work Item과 Task 관리
- Task Attempt 관리
- Owner Runtime과 Worker Supervisor 조정
- 로컬 승인
- 로컬 병합
- 실행 로그 관리
- 실패 복구

## Personal Node

사용자별 실행 환경. Owner Runtime, Worker Supervisor, Local Git Workspace, 로컬 SQLite, Inbox/Outbox, 로컬 실행 로그와 오프라인 작업 상태를 포함할 수 있다.

팀 모드의 Personal Node는 중앙 Authority와 연결되지만 중앙 DB의 복제본이나 동등한 Writer가 아니다.

`Node`는 내부 시스템 용어다. 사용자 화면에서는 가능한 한 `개인 서버`, `실행 서버`, `연결된 장치`, `이 컴퓨터` 같은 표현을 사용한다.

## Central Authority

팀 프로젝트의 공식 공유 상태를 관리하는 중앙 권위 계층.

- Project Membership
- 공유 Work Item과 Task
- Task Lease
- Scope Lock
- Approval Policy
- Decision Proposal
- Change Package
- Merge Coordinator
- Audit Event

초기 버전에서 필수 중앙 AI를 두지 않는다.

## Owner Runtime

사용자별 Personal Node에서 실행되는 Owner의 실행 환경. 사용자의 대화, 계획, 작업 분해, Worker 선택, 결과 검토와 재시도 조정을 담당한다.

Owner Runtime 내부 구조는 아직 확정하지 않는다.

## Worker Supervisor

Owner가 만든 Task Attempt를 실행할 Worker를 준비하고 실행 상태, 로그, 실패, 아티팩트를 관리하는 로컬 실행 관리자. 사용자는 Worker를 직접 조작하지 않는다.

## Local Git Workspace

개인 또는 팀 프로젝트 작업을 수행하는 로컬 Git 작업 공간. Worker 실행 결과와 테스트는 이 작업 공간에서 생성되며, 팀 모드에서는 제출 전 Change Package의 근거가 된다.

## Work Item

사람과 Owner가 보는 계획 단위. 자유로운 부모·자식 관계와 kind를 가진다.

## Task

Owner가 Worker에게 주는 실행 가능한 작은 단위.

- goal
- required_capabilities
- read_scope / write_scope / forbidden_scope
- success_criteria
- dependencies
- risk_level

## Task Attempt

Task의 실제 한 번의 실행. 재시도할 때 기존 기록을 덮어쓰지 않는다.

- attempt_id
- lease_id / lease_epoch
- assigned_node_id / assigned_worker_id
- base_commit
- status
- started_at / completed_at

## Change Package

수정되지 않는 변경 제출 기록.

- package_id
- task_attempt_id
- origin_node_id
- base_commit / head_commit
- branch_name / changed_files
- test_evidence / artifact_refs
- referenced_decisions
- signature / supersedes_package_id

## Project Admin

프로젝트 접근, 멤버, Node 연결, 정책과 비상 조치를 관리하는 인간 사용자 역할. Project Admin은 기술적 최고 책임자가 아니며 모든 코드와 설계 변경을 직접 승인하지 않는다.

- 프로젝트 사용자 초대와 제거
- Admin, Member, Viewer 역할 관리
- 개인 서버 또는 Node 연결 승인과 해제
- 승인 정책 설정
- 프로젝트 보관과 삭제
- 비상 정지
- 계정과 Node 복구
- 다른 Admin 지정

프로젝트에는 Admin을 여러 명 둘 수 있다. 단일 Project Maintainer, 영역별 Maintainer 필수 역할, `project_maintainer_id`는 검토했지만 채택하지 않는다. 관련 결정: [[07 ADR/ADR-0004 Governance and Approval Policy]]

## Member

프로젝트에 참여하여 정책이 허용하는 범위에서 작업을 제안하고 Change Package를 제출하는 기본 사용자 역할.

## Viewer

프로젝트 상태와 허용된 산출물을 조회하는 사용자 역할.

## Approval Group

특정 코드 영역이나 기술 분야의 변경을 승인할 수 있는 사용자 그룹. 예: Frontend Approvers, Backend Approvers, Database Approvers, Security Approvers, Release Approvers.

하나의 사용자가 여러 Approval Group에 속할 수 있다. 소규모 프로젝트에서는 별도 그룹 없이 사용자 승인이나 자동 승인 정책을 사용할 수 있다.

## Approval Policy

변경 파일 경로, 영향받는 영역, Task 종류, 보안 변경, 권한 상승, 데이터 손실 가능성, 비밀정보 접근, 외부 배포, 유료 API 사용, 변경 규모, 테스트 결과 등을 기준으로 필요한 승인과 테스트를 계산하는 정책.

정책 결과는 필수 테스트, Approval Group 승인 수, 사용자 명시적 승인, Owner 자동 승인 허용 여부, Merge Queue 사용 여부 등을 포함할 수 있다. 구체적인 위험도 단계와 승인 인원 수는 아직 확정하지 않는다.

## Decision Proposal

제품 방향과 설계 결정을 일반 코드 병합 승인과 분리해 다루는 제안 기록.

- 해결할 문제
- 고려한 대안
- Owner의 추천안
- 영향받는 영역
- 관련 기존 결정
- 예상 위험
- 되돌리기 비용
- 결정에 참여할 사용자 또는 Approval Group

## Merge Coordinator

중앙 Authority에서 Change Package를 수신하고 Approval Policy 계산, 승인 확인, 임시 통합, 필수 테스트, 충돌과 정책 위반 확인, Merge Queue 등록, 공식 브랜치 병합, 감사 이벤트 기록을 수행하는 역할.

Merge Coordinator는 정책 실행자이며 의미상 설계 충돌의 기술적 정답을 임의로 선택하지 않는다.
