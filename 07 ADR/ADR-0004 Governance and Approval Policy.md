---
type: adr
status: accepted
date: 2026-06-21
scope:
  - governance
  - approval
  - merge
  - authorization
---

# ADR-0004: 거버넌스와 승인 정책

## 배경

공유 프로젝트에서는 프로젝트 접근 관리, 기술 변경 승인, 제품 방향 결정, 공식 브랜치 병합이 서로 다른 책임이다. 이 책임을 한 사람에게 집중하면 작은 프로젝트에서는 단순해 보이지만, 팀 모드와 개인 모드를 모두 지원하는 구조에서는 권한 경계가 흐려진다.

이 결정은 [[07 ADR/ADR-0001 Central Authority]], [[07 ADR/ADR-0002 Personal Owner]], [[06 Security/Identity and Approval]]의 권한 분리 원칙을 확장한다. Owner 자율성, R0~R4 위험 등급, Grant와 승인 유효성의 상세 정책은 [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]에서 다룬다.

## 기존 Project Maintainer 구조의 문제점

Project Maintainer 한 명에게 다음 책임을 집중하는 구조는 검토했지만 채택하지 않는다.

- 모든 기술 결정
- 모든 코드 승인
- 모든 의미상 충돌 해결
- 모든 최종 병합
- 프로젝트 권한 관리

이 구조는 프로젝트 운영 권한과 기술 승인 권한을 혼합한다. 또한 병합과 충돌 해결을 개인의 직급 문제로 만들며, 변경 범위와 위험도에 따른 승인 정책을 표현하기 어렵다.

## 결정

프로젝트는 단일 Project Maintainer 중심 구조를 사용하지 않는다. 단일 `project_maintainer_id` 필드도 만들지 않는다. 영역별 Maintainer를 필수 프로젝트 역할로 두는 구조도 채택하지 않는다.

프로젝트 운영은 Project Admin이 담당하고, 기술 승인은 Approval Group과 Approval Policy로 계산한다. 공식 브랜치 병합은 중앙 Authority의 Merge Coordinator가 수행한다. 제품 방향과 설계 결정은 일반 코드 병합 승인과 분리하여 Decision Proposal로 다룬다.

## Project Admin

Project Admin은 프로젝트의 접근, 멤버, Node 연결, 정책과 비상 조치를 관리하는 인간 사용자 역할이다.

Project Admin의 책임:

- 프로젝트 사용자 초대와 제거
- Admin, Member, Viewer 역할 관리
- 개인 서버 또는 Node 연결 승인과 해제
- 승인 정책 설정
- 프로젝트 보관과 삭제
- 비상 정지
- 계정과 Node 복구
- 다른 Admin 지정

Project Admin은 프로젝트 전체의 기술적 최고 책임자가 아니다. 모든 코드와 설계 변경을 직접 승인하지 않는다. 프로젝트에는 Admin을 여러 명 둘 수 있다.

## Admin, Member, Viewer

사용자에게 표시되는 고정 프로젝트 역할은 다음 세 가지로 제한한다.

- Admin
- Member
- Viewer

이 역할은 프로젝트 접근과 운영 권한을 표현한다. 기술 변경 승인 권한은 고정 직급이 아니라 Approval Group과 Approval Policy로 별도 계산한다.

## Approval Group

Approval Group은 특정 코드 영역이나 기술 분야의 변경을 승인할 수 있는 사용자 그룹이다.

예:

- Frontend Approvers
- Backend Approvers
- Database Approvers
- Security Approvers
- Release Approvers

하나의 사용자는 여러 Approval Group에 속할 수 있다. Approval Group은 선택 기능이다. 소규모 프로젝트에서는 별도 그룹 없이 사용자 승인이나 자동 승인 정책을 사용할 수 있다.

## Approval Policy

필요한 승인은 고정된 전체 프로젝트 직급이 아니라 변경의 범위와 위험도에 따라 계산한다.

정책 조건 예시:

- 변경 파일 경로
- 영향받는 프로젝트 영역
- Task 종류
- 보안 변경
- 권한 상승
- 데이터 손실 가능성
- 비밀정보 접근
- 외부 배포
- 유료 API 사용
- 변경 규모
- 테스트 결과

정책 결과 예시:

- 필수 테스트
- Approval Group 승인 수
- 작성자 자신의 승인 허용 여부
- Project Admin 승인 필요 여부
- 사용자 명시적 승인
- Owner 자동 승인 허용 여부
- Merge Queue 사용 여부

구체적인 승인 인원 수와 승격 규칙은 이 ADR에서 확정하지 않는다. 위험 등급 R0~R4와 자율성 Grant 정책은 [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]에서 결정한다.

## Decision Proposal

제품 방향과 설계 결정은 일반 코드 병합 승인과 분리한다. 중요한 설계 변경이 필요하면 Owner가 Decision Proposal을 만든다.

Decision Proposal에는 다음을 포함한다.

- 해결할 문제
- 고려한 대안
- Owner의 추천안
- 영향받는 영역
- 관련 기존 결정
- 예상 위험
- 되돌리기 비용
- 결정에 참여할 사용자 또는 Approval Group

## Merge Coordinator

공식 브랜치 병합은 Project Maintainer가 직접 수행하지 않는다. 중앙 Authority의 Merge Coordinator가 다음 과정을 수행한다.

1. Change Package 수신
2. Approval Policy 계산
3. 필요한 승인 확인
4. 최신 기준 브랜치와 임시 통합
5. 필수 테스트
6. 충돌과 정책 위반 확인
7. Merge Queue 등록
8. 공식 브랜치 병합
9. 감사 이벤트 기록

Merge Coordinator는 정책 실행자이며 기술적 정답을 임의로 선택하지 않는다.

## 개인 모드 정책

개인 모드에서는 사용자 본인이 자동으로 Admin이다.

기본값:

- Owner가 Worker 작업을 관리한다.
- Worker 결과의 최종 병합은 사용자 승인이다.
- 사용자가 범위가 제한된 Grant를 부여하면 R0~R3 작업의 자동 실행을 허용할 수 있다.
- R4 작업은 자동 실행하지 않는다.
- 고위험 작업은 위험 기반 승인 UI로 Action, 대상 버전, 영향과 되돌리기 가능성을 확인한다.

개인 모드에서는 팀원 관리, Approval Group, 여러 Node 간 권한 조정 기능을 숨기거나 비활성화한다.

## 팀 모드 정책

팀 모드에서는 프로젝트 접근 권한과 기술 승인 권한을 분리한다.

- Admin은 프로젝트 운영과 정책 관리를 맡는다.
- Member는 정책이 허용하는 범위에서 작업을 제안하고 Change Package를 제출한다.
- Viewer는 프로젝트 상태와 허용된 산출물을 조회한다.
- Approval Group은 변경 영역별 승인을 제공한다.
- Approval Policy는 변경 범위와 위험도에 따라 필요한 승인과 테스트를 계산한다.
- Merge Coordinator는 승인과 테스트 결과를 확인한 뒤 공식 병합을 수행한다.

## 의미상 충돌 처리

중앙 Authority나 단일 관리자가 기술적 정답을 임의로 선택하지 않는다.

의미상 설계 충돌은 다음 절차 중 하나로 처리한다.

- 관련 Owner와 사용자의 추가 검토
- Decision Proposal 생성
- 관련 Approval Group 검토
- 실험 Task 생성
- 변경 보류
- 프로젝트에 설정된 결정 정책 실행

Project Admin은 기술적 판사가 아니라 충돌 해결 절차를 관리하는 역할이다.

## 데이터 모델 방향

단일 `project_maintainer_id` 필드는 만들지 않는다. 대신 다음 후보 테이블로 프로젝트 운영, 기술 승인, 결정 기록을 분리한다.

- project_memberships
- approval_groups
- approval_group_members
- ownership_rules
- approval_policies
- decision_proposals
- approval_decisions

정확한 스키마, 제약조건, 인덱스, 정책 평가 방식은 후속 과제로 남긴다.

## 장점

- 프로젝트 운영 권한과 기술 승인 권한을 분리한다.
- 변경 범위와 위험도에 따라 다른 승인 정책을 적용할 수 있다.
- 개인 모드와 팀 모드를 같은 개념 모델로 설명할 수 있다.
- 병합 절차와 감사 기록을 중앙 Authority에 일관되게 남길 수 있다.
- 의미상 충돌을 한 사람의 직급 문제가 아니라 명시적 결정 절차로 다룬다.

## 단점

- 단일 Maintainer 구조보다 개념과 UI가 복잡하다.
- Approval Policy 평가와 Merge Coordinator 구현이 필요하다.
- Approval Group을 사용하는 팀에서는 그룹 관리 비용이 생긴다.
- 소규모 프로젝트에서는 기본값과 숨김 처리가 충분히 단순해야 한다.

## 후속 과제

- Approval Policy DSL 또는 설정 UI 정의
- 승인 인원 수와 승격 규칙 정의
- 작성자 자기 승인 허용 조건 정의
- Project Admin 승인이 필요한 운영 작업 목록 확정
- Owner 자동 병합의 구체적인 UI와 운영 절차 정의
- 고위험 작업 분류 기준 정의
- Approval Group과 ownership_rules의 매칭 규칙 정의
- Decision Proposal 상태 전이 정의
- Merge Coordinator 실패와 재시도 정책 정의
