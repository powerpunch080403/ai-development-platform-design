# Identity and Approval

관련 결정:

- [[07 ADR/ADR-0004 Governance and Approval Policy]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]

## 분리할 신원

- Human Identity: 사용자 계정과 승인 권한
- Node Identity: 개인 서버의 장치 신원
- Worker Identity: Node 내부 Worker 실행 신원

토큰과 권한을 서로 공유하지 않는다.

## Tool 실행 경계

LLM은 등록된 Tool을 통해서만 외부 부작용을 실행한다. 파일, Git, 프로젝트 상태, Change Package, Decision Proposal, 승인 요청 같은 외부 상태 변경은 Tool Call 기록을 남긴다.

각 Tool Call에는 다음을 기록한다.

- 요청 주체
- 실행 주체
- tool_name
- requested_permissions
- approval_requirement
- idempotency_key
- 결과 또는 오류
- provider, model, prompt_version, tool_definition_version, runtime_version

민감 Tool은 Agent Run을 실패시키지 않고 승인 대기 상태로 중단한다. 사용자가 승인하면 동일 Run을 재개하고, 거부하면 거부 결과를 Owner에게 전달한다.

승인 상태에는 비밀정보를 불필요하게 직렬화하지 않는다. 승인 UI에는 인수 요약, 요청 권한, 위험도, 만료 시각, 재개에 필요한 버전 정보만 보관한다.

## 권한 분리

프로젝트 관리 권한과 기술 승인 권한을 분리한다. Project Admin은 프로젝트 접근, 멤버, Node 연결, 정책과 비상 조치를 관리하지만 프로젝트 전체의 기술적 최고 책임자는 아니다.

사용자에게 표시되는 고정 프로젝트 역할은 Admin, Member, Viewer로 제한한다. 기술 변경 승인은 고정 직급이 아니라 Approval Group과 Approval Policy로 계산한다.

## Project Admin

Project Admin의 책임:

- 프로젝트 사용자 초대와 제거
- Admin, Member, Viewer 역할 관리
- 개인 서버 또는 Node 연결 승인과 해제
- 승인 정책 설정
- 프로젝트 보관과 삭제
- 비상 정지
- 계정과 Node 복구
- 다른 Admin 지정

Project Admin은 모든 코드와 설계 변경을 직접 승인하지 않는다.

## Approval Group

Approval Group은 특정 코드 영역이나 기술 분야의 변경을 승인할 수 있는 사용자 그룹이다. 하나의 사용자는 여러 Approval Group에 속할 수 있다.

Approval Group은 선택 기능이다. 소규모 프로젝트에서는 별도 그룹 없이 사용자 승인이나 자동 승인 정책을 사용할 수 있다.

## 위험 기반 Approval Policy

필요한 승인은 변경의 범위와 위험도에 따라 계산한다.

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

구체적인 위험도 단계, 승인 인원 수, 승격 규칙은 아직 확정하지 않는다.

## Merge Coordinator

공식 브랜치 병합은 중앙 Authority의 Merge Coordinator가 수행한다.

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

## 대화로 처리

- 제품 방향
- 기술 선택
- 우선순위와 기능 범위
- 설계 대안

Owner가 Decision Proposal로 정리한다.

의미상 충돌은 관련 Owner와 사용자 검토, Decision Proposal, 관련 Approval Group 검토, 실험 Task, 변경 보류, 프로젝트에 설정된 결정 정책 중 하나로 처리한다.

## 버튼 승인

- 개인 서버 또는 연결된 장치 연결
- 권한 상승
- 비밀 접근
- 대량 삭제
- 기본 브랜치 병합
- 외부 배포
- 유료 API 사용
- 보호 영역 수정

## 개인 모드

개인 모드에서는 사용자 본인이 자동으로 Admin이다.

기본값:

- Owner가 Worker 작업을 관리한다.
- Worker 결과의 최종 병합은 사용자 승인이다.
- 사용자가 설정하면 Owner 자동 병합을 허용할 수 있다.
- 사용자가 전체 권한을 부여하면 완전 자동 실행도 허용할 수 있다.
- 고위험 작업은 별도 자동 승인 설정이 없는 한 사용자 버튼 승인이 필요하다.

개인 모드에서는 팀원 관리, Approval Group, 여러 Node 간 권한 조정 기능을 숨기거나 비활성화한다.
