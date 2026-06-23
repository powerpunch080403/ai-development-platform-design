# Identity and Approval

관련 결정:

- [[07 ADR/ADR-0004 Governance and Approval Policy]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]

## 분리할 신원

- Human Identity: 사용자 계정과 승인 권한
- Node Identity: 개인 서버의 장치 신원
- Worker Identity: Node 내부 Worker 실행 신원

토큰과 권한을 서로 공유하지 않는다.

개인 모드 MVP에서는 사용자 가입을 구현하지 않지만 Human Identity를 전역 상수나 하드코딩된 단일 ID로 대체하지 않는다. 첫 설치 시 자동 생성되는 로컬 사용자도 user 엔티티, user_id, 세션과 권한 구조를 가진다.

Primary Personal Server는 Windows와 Linux에서 같은 권한 모델을 제공해야 한다. 운영체제별 서비스 계정, 사용자 세션, 파일 권한 차이는 Runtime Platform Adapter와 설치 설계에서 분리해 다룬다.

개인 모드 MVP의 기본 접근은 localhost 또는 Tailscale이다. 인터넷에 직접 공개하거나 공개 IP와 포트포워딩을 기본으로 사용하지 않는다. Tailscale은 네트워크 접근 전제일 뿐 앱 내부 권한 모델을 대체하지 않는다.

최초 UI Client 연결에는 짧은 유효시간의 일회용 연결 코드를 사용한다. 코드는 한 번 사용하면 폐기하고, 로그에 원문을 장기 저장하지 않으며, 승인된 장치 토큰과 분리한다. 장치 토큰과 세션은 설정 화면에서 폐기할 수 있어야 한다.

프로젝트 가져오기는 허용 프로젝트 루트 안에서만 기본 허용한다. 경로 정규화, Path traversal 검사, symlink와 junction 검사, Canonical Path 비교, 저장소 유효성 검사와 dirty 상태 확인을 수행한다.

CLI 실행은 executable, argument array, working directory, environment variables, timeout, cancellation token 구조를 사용한다. 사용자 입력을 하나의 Shell 문자열로 조합하지 않는다.

비밀정보는 로그와 모델 Context에 직접 노출하지 않는다. Worktree 격리로 Worker가 사용자의 기본 작업 디렉터리를 직접 수정하지 않게 한다.

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

## 최소 권한과 자율성

기본값은 최소 권한이다. 명시되지 않은 외부 부작용은 거부한다.

- 읽기와 변경 권한을 구분한다.
- 로컬과 공유 프로젝트 권한을 구분한다.
- 개발 환경과 운영 환경 권한을 구분한다.
- 사용자는 자신이 가진 권한보다 더 큰 권한을 Owner에게 위임할 수 없다.
- Owner는 자신의 권한을 스스로 확대할 수 없다.
- Owner는 Approval Policy, 감사 로그와 보안 경계를 우회할 수 없다.

## 위험 등급

초기 위험 등급:

- R0 관찰: 외부 상태를 변경하지 않는 읽기 전용 행동. 기본 자동 허용.
- R1 로컬·가역 변경: 개인 작업공간 안에서 쉽게 되돌릴 수 있고 공유 상태에 직접 영향을 주지 않는 변경.
- R2 공유 프로젝트 변경 제안: 중앙 검증과 병합 전에는 공식 상태가 되지 않는 공유 프로젝트 변경 제안.
- R3 고영향 행동: 공식 상태, 외부 환경, 비용, 권한 또는 중요한 데이터에 직접 영향을 주는 행동.
- R4 치명적·보안 관리 행동: 복구가 매우 어렵거나 권한, 감사 가능성, 중요한 운영 데이터에 영향을 주는 행동.

하나의 행동이 여러 조건에 해당하면 더 높은 위험 등급을 적용한다. 최종 위험도는 Policy Engine이 계산한다.

## 자율성 프로필

기본 프로필:

- Ask for approval (구 Confirm Every Change): R0만 자동, R1 이상 사용자 승인.
- Approve on my behalf (구 Allow Local Work): R0, R1 자동, R2 이상 승인 또는 프로젝트 정책 적용.
- Full access (구 Trusted Owner): R0, R1, 허용된 범위의 R2 자동, 사전 위임된 R3 자동 가능, R4는 항상 인간 승인.
- Custom (구 Autonomous): 사용자가 가진 권한 중 명시적으로 위임한 R0~R3 작업 자동 가능, R4는 자동 수행하지 않음.

자율성 프로필은 정책 템플릿이며 실제 권한은 범위가 있는 Owner Grant로 표현한다.

## 정책 우선순위

여러 정책이 충돌하면 더 제한적인 규칙을 우선한다.

1. 플랫폼의 변경 불가능한 보안 규칙
2. 중앙 Authority 또는 Workspace 정책
3. 프로젝트 Approval Policy
4. 사용자의 Owner Grant
5. Agent Run에서 요청한 권한
6. Tool Call이 요청한 권한

상위 정책이 거부한 행동을 하위 정책이 허용할 수 없다. 팀 정책과 다른 사용자의 권한은 개인 Grant로 우회할 수 없다.

## 승인 유효성

승인은 요청 당시의 Action과 대상 버전에만 유효하다.

승인 후 Tool arguments, 대상 파일 또는 리소스, 기준 commit, 배포 환경, 예상 비용, 위험 등급, Approval Policy, 승인 대상 버전, 실행 권한이 변경되면 stale 상태로 처리하고 재승인을 요구할 수 있다.

승인된 Action을 다른 Action에 재사용하지 않는다.

## 비밀정보 처리

Owner와 Tool은 필요한 비밀 원문을 모델 Context에 무조건 전달하지 않는다.

- 비밀은 가능하면 Tool 실행 환경에서만 주입한다.
- 모델에는 비밀 이름과 사용 가능 여부만 제공한다.
- Tool 결과, 로그, 이벤트에서 비밀 문자열을 제거하거나 마스킹한다.
- 비밀 원문 조회, 복사 또는 외부 전송은 높은 위험 행동으로 분류한다.

## 긴급 중단과 감사

사용자는 현재 Agent Run 취소, 새 Tool Call 차단, Worker 실행 중지 요청, Owner Grant 철회, Autonomous 모드 해제, Personal Node 연결 해제를 수행할 수 있어야 한다.

다음 항목은 감사 가능해야 한다.

- 누가 권한을 위임했는가
- 어떤 Owner와 Run이 Action을 요청했는가
- 어떤 Tool이 실행했는가
- 위험 등급과 산정 이유
- 어떤 정책이 적용됐는가
- 누가 승인 또는 거부했는가
- 어떤 Grant가 사용됐는가
- 실제 결과, 실패, 재시도, 권한 철회

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

구체적인 승인 인원 수와 승격 규칙은 아직 확정하지 않는다. 위험 등급 R0~R4와 기본 정책은 [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]에서 정의한다.

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
- 사용자가 범위가 제한된 Grant를 부여하면 R0~R3 작업의 자동 실행을 허용할 수 있다.
- R4 작업은 자동 실행하지 않는다.
- 고위험 작업은 위험 기반 승인 UI로 Action, 대상 버전, 영향과 되돌리기 가능성을 확인한다.

개인 모드에서는 팀원 관리, Approval Group, 여러 Node 간 권한 조정 기능을 숨기거나 비활성화한다.
