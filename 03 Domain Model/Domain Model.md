# Domain Model

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
