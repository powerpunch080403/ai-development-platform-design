---
type: adr
status: accepted
date: 2026-06-21
scope:
  - autonomy
  - approval
  - risk
  - authorization
  - security
---

# ADR-0007: 자율성과 승인 위험 정책

## 배경

[[07 ADR/ADR-0004 Governance and Approval Policy]]는 프로젝트 운영 권한과 기술 승인 권한을 분리했다. [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]는 Owner가 등록된 Tool Call을 통해 외부 부작용을 실행하고, 필요한 경우 Agent Run을 승인 대기 상태로 중단한다고 결정했다. 개인 모드 MVP의 기본 프로필과 승인 화면 범위는 [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]에서 구체화한다.

이제 Owner에게 어느 정도 자율성을 줄 수 있는지, 어떤 행동이 승인을 요구하는지, 개인 모드와 팀 모드에서 승인 원본을 누가 소유하는지 결정해야 한다.

## 문제

Owner의 자율성을 모델 자체에 부여한 무제한 권한으로 해석하면 다음 문제가 생긴다.

- 사용자가 가진 권한보다 큰 권한을 Owner가 행사할 수 있다.
- 팀 프로젝트의 Approval Policy와 다른 사용자의 권한을 우회할 수 있다.
- 파일 수정, 배포, 비용 발생, 비밀정보 접근, 권한 변경이 같은 승인 수준으로 취급된다.
- 승인된 Action이 다른 대상이나 변경된 버전에 재사용될 수 있다.
- 감사 로그와 정책 평가 근거가 남지 않는다.

따라서 자율성은 범위가 있는 Grant와 위험 기반 Approval Policy로 표현해야 한다.

## 결정

Owner의 자율성은 모델 자체에 부여하는 무제한 권한이 아니다. Owner는 명시적으로 등록된 Tool과 사용자가 부여한 권한 범위 안에서만 행동한다.

기본 원칙:

- 기본값은 최소 권한
- 명시되지 않은 외부 부작용은 거부
- 읽기와 변경 권한을 구분
- 로컬과 공유 프로젝트 권한을 구분
- 개발 환경과 운영 환경 권한을 구분
- 사용자는 자신이 가진 권한보다 더 큰 권한을 Owner에게 위임할 수 없음
- Owner는 자신의 권한을 스스로 확대할 수 없음
- Owner는 Approval Policy, 감사 로그와 보안 경계를 우회할 수 없음

승인 필요 여부는 행동 종류, 변경 대상, 변경 범위, 실행 환경, 되돌릴 수 있는지 여부, 보안 영향, 데이터 손실 가능성, 비용 발생 가능성, 외부 시스템 영향, 프로젝트 Approval Policy, 사용자 또는 Admin이 Owner에게 위임한 권한, 현재 Agent Run 권한을 조합해 결정한다.

## 행동 기반 승인

승인은 LLM이 생성한 자연어 답변 전체에 적용하지 않는다. 실제 외부 부작용을 발생시키는 Tool Call 또는 Action에 적용한다.

승인 대상 예:

- 파일 수정
- 명령 실행
- Task 생성
- Worker 실행
- Git commit
- 공식 브랜치 병합
- 배포
- 비밀정보 접근
- 권한 변경
- 유료 API 호출
- 외부 메시지 전송
- 데이터 삭제

하나의 Agent Run 안에서도 Tool Call마다 서로 다른 승인 정책이 적용될 수 있다.

### Owner와 Worker의 승인 책임 분리

- User approval is requested by Owner, not Worker. Worker는 사용자에게 직접 승인 요청하지 않는다.
- Worker execution may require approval depending on write_scope breadth, protected paths, danger flag, command execution, external access, or system-level permissions.
- danger permission skip (`--dangerously-skip-permissions` 등) is not automatically allowed; it is controlled by local config, Owner Grant, and policy.
- A broad user request to Owner does not grant unlimited Worker authority. 사용자가 넓은 자연어 권한을 줬더라도 Owner가 Task를 나눌 때 risk level, write_scope, danger flag 필요성 등을 통제하고 승인을 거쳐야 한다.

## 위험 등급 R0~R4

초기 위험 등급은 R0부터 R4까지 다섯 단계로 정의한다. 구체적인 파일 수, 비용, 데이터 크기와 같은 수치 기준은 후속 정책 설정으로 남긴다.

### R0 - 관찰

외부 상태를 변경하지 않는 읽기 전용 행동.

예:

- 프로젝트 정보 조회
- 코드 읽기
- Git status 조회
- 로그 조회
- 테스트 결과 조회
- 문서 검색
- 계획 작성

기본 정책:

- 자동 허용
- 읽기 권한 범위는 확인
- 민감한 비밀 원문은 별도 보안 정책 적용

### R1 - 로컬·가역 변경

개인 작업공간 안에서 쉽게 되돌릴 수 있고 공유 상태에 직접 영향을 주지 않는 변경.

예:

- 개인 작업 브랜치 파일 수정
- 새 파일 생성
- 로컬 테스트 실행
- 포맷터와 린터 실행
- 로컬 임시 브랜치 생성
- 로컬 commit 생성
- 개인 Task 초안 생성

기본 정책:

- Owner 자동 실행 허용 가능
- 기본 사용자 설정에 따라 자동 또는 승인
- 삭제나 대규모 변경은 더 높은 위험으로 승격

### R2 - 공유 프로젝트 변경 제안

공유 프로젝트에 영향을 줄 수 있지만 중앙 검증과 병합 전에는 공식 상태가 되지 않는 행동.

예:

- 팀 Task 생성 제안
- Change Package 제출
- Decision Proposal 생성
- 중앙 Merge Queue 등록 요청
- 공유 Work Item 수정 제안
- 비보호 영역 변경 제출

기본 정책:

- 프로젝트 정책과 Owner 권한을 확인
- 자동 제출을 허용할 수 있음
- 공식 병합이나 고위험 영향은 별도 등급으로 평가

### R3 - 고영향 행동

공식 상태, 외부 환경, 비용, 권한 또는 중요한 데이터에 직접 영향을 주는 행동.

예:

- 공식 브랜치 병합
- 운영 환경 배포
- 데이터베이스 스키마 변경
- 유료 API 또는 클라우드 자원 생성
- 비밀정보 사용
- 외부 사용자에게 메시지 전송
- 보호 영역 수정
- 서비스 재시작
- 프로젝트 설정 변경

기본 정책:

- 사용자 명시적 승인
- 사용자가 사전에 제한된 범위로 위임한 경우 자동 실행 가능
- 팀 모드에서는 Approval Policy가 사용자 위임보다 우선
- 실행 직전에 대상과 버전을 다시 검증

### R4 - 치명적·보안 관리 행동

복구가 매우 어렵거나 다른 사용자의 권한, 감사 가능성, 중요한 운영 데이터에 영향을 주는 행동.

예:

- 대규모 또는 복구 불가능한 데이터 삭제
- 프로젝트 Admin 권한 부여 또는 제거
- Approval Policy 약화
- 감사 로그 비활성화 또는 삭제
- 보호 브랜치 강제 Push
- 백업 삭제
- 운영 비밀 원문 외부 전송
- 보안 정책 해제
- 다른 사용자의 개인 Node 제어
- Owner 자신의 권한 확대
- 프로젝트 전체 삭제

기본 정책:

- 자동 승인 불가
- 명시적 인간 승인 필요
- 팀 정책에 따라 다중 승인 가능
- 실행 전 재인증 또는 추가 확인을 요구할 수 있음
- Owner 전체 권한 모드에서도 우회할 수 없음

## 위험도 승격

하나의 행동이 여러 조건에 해당하면 더 높은 위험 등급을 적용한다.

다음 조건은 위험도를 높일 수 있다.

- 운영 환경
- 보호된 경로
- 데이터 손실 가능성
- 외부 네트워크 전송
- 비밀정보 포함
- 비용 발생
- 넓은 파일 또는 시스템 범위
- 다른 사용자의 자원
- 되돌리기 어려움
- 오래된 승인 또는 변경된 대상 버전

Owner 또는 Tool 구현이 위험도를 낮춰 보고할 수 없도록 최종 위험도는 Policy Engine이 계산한다.

## 자율성 프로필

사용자에게 다음 네 가지 기본 프로필을 제공한다.

### Confirm Every Change

- R0만 자동
- R1 이상은 사용자 승인
- 처음 사용하는 사용자에게 가장 보수적인 설정

### Allow Local Work

- R0, R1 자동
- R2 이상 승인 또는 프로젝트 정책 적용
- 기본 개인 모드 권장 프로필

### Trusted Owner

- R0, R1, 허용된 범위의 R2 자동
- 사전 위임된 R3 행동 자동 가능
- R4는 항상 인간 승인
- 프로젝트, 경로, 환경, 비용과 기간으로 권한 제한

### Autonomous

- 사용자가 가진 권한 중 명시적으로 위임한 R0~R3 작업을 Owner가 자동 수행
- R4는 자동 수행하지 않음
- 팀 모드에서는 중앙 Approval Policy와 다른 사용자의 권한이 우선
- 모든 행동은 감사 기록
- 즉시 중단 기능 제공

자율성 프로필은 편의를 위한 정책 템플릿이며 실제 권한은 세부 Grant로 표현한다.

## 권한 위임 Grant

Owner에게 주는 권한은 단순한 `full_access = true`가 아니라 범위가 있는 Grant로 저장한다.

Grant 후보 필드:

- grant_id
- granted_by_user_id
- owner_id
- project_id
- allowed_actions
- allowed_tools
- allowed_scopes
- allowed_environments
- maximum_risk_level
- budget_limit
- valid_from
- expires_at
- allow_background_execution
- allow_external_network
- allow_secret_use
- created_at
- revoked_at

Grant는 다음 단위로 줄 수 있다.

- 이번 한 번
- 현재 Agent Run
- 현재 Conversation
- 일정 시간
- 특정 프로젝트
- 특정 경로 또는 영역
- 특정 Tool
- 특정 환경

Owner는 Grant를 생성, 확대, 연장하거나 취소 철회 상태를 변경할 수 없다.

## 정책 우선순위

여러 정책이 충돌하면 더 제한적인 규칙을 우선한다.

우선순위:

1. 플랫폼의 변경 불가능한 보안 규칙
2. 중앙 Authority 또는 Workspace 정책
3. 프로젝트 Approval Policy
4. 사용자의 Owner Grant
5. Agent Run에서 요청한 권한
6. Tool Call이 요청한 권한

상위 정책이 거부한 행동을 하위 정책이 허용할 수 없다.

팀 프로젝트에서 개인 사용자가 Owner에게 전체 권한을 주더라도 다른 사용자의 자원, Project Admin 권한과 중앙 정책을 우회할 수 없다.

## 승인 요청

승인이 필요한 Action은 Approval Request를 생성하고 Agent Run을 `waiting_for_approval` 상태로 전환한다.

Approval Request에는 다음 정보를 포함한다.

- approval_id
- run_id
- tool_call_id
- 요청한 행동
- 대상 프로젝트와 환경
- 대상 리소스
- 변경 범위 요약
- 위험 등급
- 위험 등급 산정 이유
- 예상 영향
- 되돌리기 방법
- 비용 또는 외부 영향
- 사용될 권한
- 요청 시점의 대상 버전
- 만료 시각
- 승인 가능한 사용자 또는 Approval Group

사용자 UI에서는 단순히 "허용할까요?"만 표시하지 않는다.

최소한 다음을 보여준다.

- 무엇을 하는가
- 어디에 적용되는가
- 무엇이 바뀌는가
- 왜 필요한가
- 위험은 무엇인가
- 되돌릴 수 있는가
- 한 번만 허용할지 범위로 허용할지

## 승인 선택지

승인 UI는 상황에 따라 다음 선택지를 제공할 수 있다.

- 거부
- 이번 한 번 허용
- 현재 Run 동안 허용
- 이 프로젝트의 지정 범위에서 허용
- 제한된 기간 동안 허용
- 설정 화면에서 세부 권한 검토

R4 행동에는 지속적인 자동 승인 선택지를 제공하지 않는다.

## 승인 유효성과 stale 처리

승인은 요청 당시의 Action과 대상 버전에만 유효하다.

승인 후 다음이 변경되면 재승인을 요구할 수 있다.

- Tool arguments
- 대상 파일 또는 리소스
- 기준 commit
- 배포 환경
- 예상 비용
- 위험 등급
- Approval Policy
- 승인 대상 버전
- 실행에 사용되는 권한

승인된 Action을 다른 Action에 재사용하지 않는다. 승인이 만료되거나 대상 상태가 변경되면 stale 상태로 처리한다.

## 개인 모드 정책

개인 모드에서는 사용자가 자신의 Owner에 대해 자율성 프로필을 선택한다.

기본 권장 설정:

- R0 자동
- R1 자동
- R2 사용자 설정에 따라 자동 또는 승인
- R3 사용자 승인
- R4 항상 명시적 승인

사용자는 Trusted Owner 또는 Autonomous 프로필을 선택해 R0~R3 권한을 제한된 범위로 미리 위임할 수 있다.

개인 모드의 사용자가 "전체 권한"을 선택하더라도 다음은 유지한다.

- 감사 기록
- Owner의 자기 권한 확대 금지
- 승인 정책 자체를 Owner가 약화하는 것 금지
- R4 자동 실행 금지
- 즉시 중단과 권한 철회 기능

## 팀 모드 정책

팀 모드에서 개인 사용자의 Owner Grant는 해당 사용자가 가진 프로젝트 권한을 넘을 수 없다.

기본 원칙:

- 개인 Owner가 로컬 작업을 수행하는 것은 사용자 Grant 적용
- 중앙 공식 상태 변경은 프로젝트 Approval Policy 적용
- 보호 영역은 관련 Approval Group 정책 적용
- Project Admin 또는 다중 승인이 필요한 Action은 개인 Grant로 우회 불가
- 공식 승인 결과의 원본은 중앙 Authority가 소유
- Personal Node의 승인 상태는 표시와 재개를 위한 캐시 또는 대기 상태
- 다른 사용자의 개인 Owner나 Personal Node를 제어할 수 없음

## 비밀정보 처리

Owner와 Tool은 필요한 비밀 원문을 모델 Context에 무조건 전달하지 않는다.

가능하면 다음 방식을 사용한다.

- 비밀은 Tool 실행 환경에서만 주입
- 모델에는 비밀 이름과 사용 가능 여부만 제공
- Tool 결과에서 비밀 문자열 제거
- 로그와 이벤트에서 마스킹
- 외부 네트워크 전송 전에 정책 확인

비밀 원문 조회, 복사 또는 외부 전송은 별도의 높은 위험 행동으로 분류한다.

## 비용과 외부 자원

유료 API, 클라우드 인스턴스, 배포 환경 등 비용을 발생시키는 행동은 Budget Grant 안에서만 자동 실행할 수 있다.

기록 후보:

- 예상 비용
- 실제 비용
- 기간별 누적 비용
- 예산 한도
- 초과 시 승인 필요

정확한 금액 기준과 통화 정책은 후속 설계로 남긴다.

## 긴급 중단

사용자는 언제든 다음을 수행할 수 있다.

- 현재 Agent Run 취소
- Owner의 새 Tool Call 차단
- 모든 Worker 실행 중지 요청
- Owner Grant 철회
- Autonomous 모드 해제
- Personal Node를 중앙 Authority에서 연결 해제

이미 외부 시스템에서 완료된 부작용은 단순 취소로 되돌아가지 않을 수 있으므로 보상 또는 복구 절차를 별도로 안내한다.

## 감사

다음 항목은 감사 가능해야 한다.

- 누가 권한을 위임했는가
- 어떤 Owner와 Run이 Action을 요청했는가
- 어떤 Tool이 실행했는가
- 위험 등급과 산정 이유
- 어떤 정책이 적용됐는가
- 누가 승인 또는 거부했는가
- 어떤 Grant가 사용됐는가
- 실제 결과
- 실패와 재시도
- 권한 철회

개인 모드에서도 최소 로컬 감사 기록을 유지한다.

## 데이터 모델 방향

개인 모드 또는 Personal Node의 SQLite 후보:

- autonomy_profiles
- owner_grants
- approval_requests
- approval_decisions
- policy_evaluations
- grant_revocations
- security_audit_events
- budget_usage

팀 Central Authority의 PostgreSQL 후보:

- approval_policies
- approval_groups
- approval_requests
- approval_decisions
- owner_grants 또는 central_grants
- policy_evaluations
- grant_revocations
- security_audit_events
- budget_usage

개인 프로젝트의 Approval Request와 결과, 개인 Owner Grant, 개인 자율성 설정, 로컬 감사 기록은 Local Control Plane 또는 Personal Node가 원본이다.

팀 프로젝트의 Approval Policy, Approval Group, 공식 Approval Request와 결과, 공식 Grant 또는 중앙 위임, 팀 감사 기록은 중앙 Authority가 원본이다. Personal Node의 팀 승인 데이터는 캐시 또는 실행 재개 상태이며 공식 원본이 아니다.

구체적인 최종 스키마는 후속 설계로 남긴다.

## 장점

- Owner 자율성을 무제한 권한이 아니라 명시적 Grant로 표현한다.
- Tool Call 단위로 실제 외부 부작용을 승인할 수 있다.
- R0~R4 위험 등급으로 기본 정책을 설명할 수 있다.
- 개인 모드의 편의성과 팀 모드의 중앙 정책을 함께 지원한다.
- stale 승인, 권한 철회, 감사 기록을 정책의 일부로 다룬다.

## 단점

- 단순한 "항상 묻기" 또는 "전체 자동" 설정보다 개념이 많다.
- Policy Engine과 위험도 산정 근거 저장이 필요하다.
- 사용자 UI가 승인 이유와 범위를 충분히 설명해야 한다.
- 팀 모드에서는 중앙 정책과 개인 Grant의 충돌 처리가 필요하다.

## 후속 과제

- 각 위험 등급의 정량 임계값 정의
- 비용 예산의 단위와 기본값 정의
- 재인증이 필요한 Action 목록 정의
- 팀 다중 승인 수와 승격 규칙 정의
- 보호 영역을 정의하는 UI와 파일 형식 정의
- Emergency Stop의 프로세스 종료 범위 정의
- 권한 철회와 이미 실행 중인 Tool Call의 경합 처리 정의
