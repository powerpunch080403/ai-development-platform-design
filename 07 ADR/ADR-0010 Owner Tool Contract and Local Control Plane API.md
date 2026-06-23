---
type: adr
status: accepted
date: 2026-06-22
scope:
  - personal-mode
  - owner-runtime
  - tool-contract
  - local-control-plane
  - ui-shell
  - http-api
  - desktop-app
  - policy
  - approval
  - worker
  - artifact
  - audit
---

# ADR-0010: Owner Tool Contract와 Local Control Plane API

## 배경

[[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]는 Owner의 외부 부작용을 Tool Call로 기록하고 재개 가능한 Agent Run에서 실행하도록 결정했다. [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]는 Owner Grant, R0~R4 위험 등급, 승인과 stale 처리를 정했다. [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]는 브라우저 기반 첫 UI와 Primary Personal Server를 채택했고, [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]는 Owner가 SQLite, Git, 파일시스템과 Worker를 직접 조작하지 않고 서버 Tool Call을 통해서만 수행하도록 경계를 확정했다.

이 ADR은 그 결정을 뒤집지 않고 Owner Tool Contract, Local Control Plane의 application service 경계, UI Shell과 local transport의 관계를 구체화한다.

## 문제

Tool, HTTP API와 UI를 같은 개념으로 취급하거나 Owner가 서버 내부 객체를 직접 호출하게 두면 다음 문제가 생긴다.

- 브라우저 UI가 최종 제품 형태로 굳어 Desktop App 전환이 어려워진다.
- Tool Contract가 HTTP endpoint에 결합되어 IPC, background queue와 팀 모드 API에서 재사용하기 어렵다.
- 일반 AI 능력과 플랫폼이 통제해야 하는 상태 변경 기능의 경계가 흐려진다.
- 권한 평가, 승인, idempotency와 감사가 호출 경로마다 다르게 구현될 수 있다.
- 같은 Task Attempt를 여러 Worker가 동시에 실행할 수 있다.
- 큰 로그와 산출물을 SQLite에 직접 저장해 실행 상태 저장소가 비대해질 수 있다.
- Personal Node의 로컬 기록과 팀 공식 중앙 기록의 소유권이 혼합될 수 있다.

## 결정

- 공식 장기 UI 목표는 Desktop App이다. MVP는 Desktop App Shell에 재사용 가능한 browser-accessible Web UI를 먼저 구현한다.
- MVP의 첫 local transport는 localhost HTTP API다. HTTP API는 브라우저 전용이 아니며 앱 그 자체도 아니다.
- Owner Runtime은 Internal Tool Contract를 통해 Local Control Plane의 application service를 호출한다. 같은 서버 내부 호출에 HTTP를 강제하지 않는다.
- Tool은 일반 AI 능력 확장이 아니라 플랫폼이 소유한 상태와 부작용을 안전하게 통제하는 서버 기능이다.
- Tool Registry, 공통 Tool Call Envelope, Tool Boundary와 Local Policy Engine을 둔다.
- 승인 대기는 실패가 아니라 Approval Interruption과 Agent Run의 `waiting_for_approval` 상태로 기록한다.
- Personal Mode MVP에 Worker lease/claim과 `artifact_refs`를 포함한다.
- command policy는 고정된 세밀한 명령 allowlist가 아니라 Owner Grant, Autonomy Profile과 Project Settings를 중심으로 표현한다.
- Personal Mode의 squash merge 승인 필요 여부는 정책으로 평가한다. 기본 `Allow Local Work`에서는 사용자 승인이 필요하며, 사용자가 더 넓은 Grant를 명시한 범위에서는 자동 squash merge를 허용할 수 있다. R4 자동 실행 금지는 유지한다.

## UI Shell과 Local Transport

제품 구조를 특정 UI 실행 방식에 고정하지 않는다.

```text
UI Shell
→ Local Transport
→ Primary Personal Server / Local Control Plane
→ SQLite / Git / Worker Supervisor / Artifact Store
```

초기 MVP 형태:

```text
Browser-accessible Web UI
→ localhost HTTP API
→ Primary Personal Server
→ Local Control Plane
```

장기 제품 형태:

```text
Desktop App Shell
→ local HTTP API 또는 IPC
→ Primary Personal Server / Local Control Plane
```

UI 기술은 Desktop-ready Web UI로 설계한다. 정확한 표현은 “브라우저 앱을 최종 제품으로 만든다”가 아니라 “Desktop App에 재사용 가능한 Web UI를 먼저 만든다”이다. 브라우저 접속은 MVP 개발, 디버그와 원격 접속 수단으로 유지할 수 있다.

MVP에서는 localhost HTTP API를 우선한다. 향후 같은 Web UI를 WebView, Tauri, Electron류 Desktop App Shell에서 재사용하고 local HTTP API 또는 IPC로 통신할 수 있다. Desktop App Shell 기술, local IPC, 패키징, 자동 업데이트와 OS keychain 연동은 후속 결정이다.

## Tool의 정의

```text
Tool = Owner가 서버 내부 상태, 프로젝트, Worker, Git, 승인과 실행 기록을
       안전하게 조작하기 위해 반드시 서버를 통해 호출해야 하는 기능
```

Tool은 AI의 일반 능력을 확장하는 범용 플러그인 목록이 아니다. 검색, 이미지 생성, 일반 웹 탐색과 일반 코드 작성처럼 모델이나 외부 제품이 이미 수행하는 능력을 플랫폼 서버 Tool로 중복 제공하지 않는다. 플랫폼 Tool의 핵심 범위는 플랫폼이 소유한 상태와 부작용이다.

```text
AI가 인터넷 검색을 한다
→ 플랫폼 서버 Tool의 핵심 범위 아님

AI가 SQLite에 Task를 만든다
→ 반드시 서버 Tool

AI가 Worker를 실행한다
→ 반드시 서버 Tool

AI가 Git worktree를 만들거나 squash merge를 준비한다
→ 반드시 서버 Tool

AI가 승인 요청을 만들거나 승인 결과를 기록한다
→ 반드시 서버 Tool
```

## Tool Contract와 HTTP API의 관계

Internal Tool Contract를 먼저 정의한다. HTTP API는 UI Shell, 외부 클라이언트와 향후 다른 Node가 동일한 application service layer를 호출하는 transport 중 하나다. Tool Contract를 HTTP endpoint와 1:1로 고정하지 않는다.

```text
Owner Agent Run
→ Internal Tool Contract
→ Local Control Plane
→ SQLite / Git / Worker Supervisor / Artifact Store
```

Primary Personal Server는 Web UI 제공, Local Control Plane, Owner Runtime, Worker Supervisor, SQLite, Git과 Artifact Store를 실행한다. Owner Runtime이 같은 서버 안에서 실행될 때는 내부 Tool Contract를 직접 사용할 수 있으며 HTTP를 거칠 필요가 없다. HTTP API는 UI Shell과 서버 기능을 연결하는 통신 창구다.

> Tool Contract는 서버 내부 업무 처리 규칙이고, HTTP API는 그 기능을 UI Shell이나 다른 Node가 호출하기 위한 외부 창구다.

Tool Contract provides bounded capabilities. It does not define keyword command routing from user messages. Owner Agent chooses tools; Local Control Plane enforces boundaries. (See ADR-0019)

**Owner Runtime Provider**:
Owner Runtime Providers, including Codex CLI and future API providers, may request Tool Calls but must not bypass the Tool Contract or Local Control Plane.

이 분리로 Desktop App, local IPC, background queue와 Central Authority API가 추가되어도 Tool Contract와 application service 규칙을 재사용할 수 있다.

## Tool Registry

ADR-0010은 최종 Tool 전체 목록을 고정하지 않는다. Tool Registry를 두고 Tool의 추가, 삭제, 비활성화, deprecation과 버전 변경을 관리한다.

Tool Definition 후보 속성:

```text
tool_name
tool_version
category
description
input_schema_ref
output_schema_ref
has_side_effect
default_risk_level
required_grants
scope_evaluator
idempotency_required
approval_behavior
audit_required
enabled
deprecated_at
```

MVP 초기 Tool 후보는 다음과 같으며 최종 고정 목록이 아니다.

```text
project.create
project.update_settings
repository.register
repository.check_dirty
repository.get_status
repository.get_diff
conversation.create
message.append
agent_run.create
task.create
task.update_status
task_attempt.create
worker.register
worker.claim_attempt
worker.heartbeat
worker.release_claim
worker.start_run
worker.cancel_run
worker.submit_result
worktree.create
worktree.cleanup
approval.request
approval.record_decision
merge.prepare_squash
merge.perform_squash
artifact.create_ref
audit.record_event
```

## Tool Call Envelope

읽기 전용 호출을 포함한 모든 Tool Call은 추적, 감사, 재시작 복구, 승인 대기, 실패 분석, idempotency와 팀 모드 연결을 위한 공통 Envelope를 가진다.

```text
tool_call_id
tool_name
tool_version
tool_category
caller_type
caller_id
user_id
device_id
conversation_id
agent_run_id
agent_run_step_id
project_id
repository_id
work_item_id
task_id
task_attempt_id
worker_run_id
risk_level
idempotency_key
correlation_id
arguments
status
result_ref
error_code
error_message
created_at
started_at
completed_at
```

`caller_type` 후보:

```text
owner
ui
worker
system
central_authority
```

상태 후보:

```text
created
policy_checking
waiting_for_approval
running
succeeded
failed
cancelled
skipped_duplicate
```

부작용 Tool은 `idempotency_key`가 필수다. 같은 key와 동일한 action scope의 호출이 이미 성공했다면 복구 중 다시 실행하지 않고 기존 결과를 반환하거나 `skipped_duplicate`로 기록한다. 큰 결과는 SQLite 본문 대신 `result_ref` 또는 `artifact_ref`로 연결한다.

## Tool Boundary

Tool Boundary는 단순 권한 목록이 아니라 플랫폼의 정상 실행 틀을 정의한다.

- Owner는 Tool Contract 밖에서 서버 상태를 변경할 수 없다.
- Worker는 Task Attempt와 Worktree 생명주기 밖에서 작업할 수 없다.
- Merge는 Merge Tool과 Approval/Grant 흐름 밖에서 수행할 수 없다.
- UI는 SQLite, Git 또는 파일시스템을 직접 변경해 서버 상태를 우회할 수 없다.
- 모든 상태 변경은 Local Control Plane 또는 Personal Server가 소유한 application service layer를 통한다.

금지 구조:

```text
Owner LLM → SQLite 직접 수정
Owner LLM → git merge 직접 실행
Owner LLM → 파일시스템 직접 변경
Owner LLM → Worker 직접 실행
Worker → Task 없이 임의 실행
Worker → 등록되지 않은 경로 직접 수정
UI → 서버 상태 우회 변경
```

허용 구조:

```text
Owner LLM
→ Tool Call
→ Local Control Plane
→ Tool Boundary 검사
→ Local Policy Engine
→ SQLite / Git / File System / Worker Supervisor / Artifact Store
→ Runtime Event / Audit Event 기록
```

평가 순서:

1. 행동이 Tool Contract 안의 정상 행동인지 확인한다.
2. 호출자가 이 행동을 할 권한이 있는지 평가한다.
3. 승인이 필요한지 평가한다.
4. 실행 결과와 감사 기록을 남긴다.

## Local Policy Engine

Personal Mode에는 Central Authority가 없으므로 Primary Personal Server 내부의 Local Policy Engine이 정책을 평가한다.

Tool Boundary는 행동이 플랫폼의 정상 실행 틀에 속하는지 판단한다. Local Policy Engine은 호출자의 행동을 허용할지, 승인 대기로 보낼지, 거절할지를 판단한다. 평가 입력에는 Owner Grant, Autonomy Profile, Project Settings, Session/Device 권한, Tool Registry와 Risk Level이 포함된다.

평가 예:

```text
이 Owner가 이 프로젝트에서 파일을 수정할 수 있는가?
이 Tool은 R1인가 R3인가?
이 작업은 사용자 승인이 필요한가?
이 경로는 허용된 project/worktree root 안인가?
이 merge는 현재 Autonomy Profile에서 자동 가능한가?
이 Tool은 현재 Owner Grant 범위 안에 있는가?
```

## Approval Interruption

Tool Call이 승인 필요로 평가되면 실패로 처리하지 않는다. Approval Interruption을 만들고 Agent Run을 `waiting_for_approval`로 전환하며 UI Shell에 승인 대기 카드를 표시한다.

```text
Owner calls Tool
→ Tool Boundary check
→ Local Policy Engine
→ approval required
→ Approval Interruption created
→ Agent Run waiting_for_approval
→ User approves or rejects
→ Tool resumes or Run handles rejection
```

사용자가 승인하면 같은 Agent Run과 Tool Call을 안전하게 재개한다. 거절하면 Owner가 대안 요청, 변경 범위 축소, 취소 또는 실패를 처리한다. 승인 뒤 arguments, repository, base commit, risk level 또는 변경 범위가 달라지면 기존 승인을 stale로 처리할 수 있어야 한다.

## Worker Lease와 Claim

Personal Mode MVP에서도 Worker는 Task 없이 독립 실행하지 않으며 Task Attempt를 claim한 뒤 실행한다. 개인 모드에서 여러 Worker를 등록할 가능성과 서버 재시작, Worker 종료, 연결 끊김과 취소를 안전하게 처리하기 위해 lease를 둔다.

- 하나의 active Task Attempt에는 동시에 하나의 유효한 claim만 허용한다.
- claim에는 만료 시각과 lease 식별자가 있다.
- Worker는 heartbeat로 lease를 갱신한다.
- lease가 만료되면 Attempt는 재시도 가능한 상태가 될 수 있다.
- 결과 제출, 취소 또는 명시적 release로 lease를 종료한다.

상태 후보:

```text
available
claimed
running
heartbeat_lost
expired
released
completed
cancelled
failed
```

v1의 Worker package/report 상세 형식을 그대로 채택하지 않는다. ADR-0009의 Task Attempt와 Worker Run 모델에 맞춘 상세 계약은 후속 설계로 남긴다.

## Artifact Reference

Artifact는 diff patch, test/lint/build log, Worker report, error log, CLI transcript, screenshot, coverage report와 생성 문서 같은 실행 증거물이다. 큰 파일은 Artifact Store에 저장하고 SQLite에는 `artifact_ref`를 기록한다.

Artifact는 Agent Run, Tool Call, Worker Run과 Task Attempt에 연결되어 Owner 검토, 사용자 리뷰, 실패 분석과 재시도에 쓰인다.

ArtifactRef 후보 속성:

```text
artifact_id
owner_type
owner_id
project_id
repository_id
task_attempt_id
worker_run_id
tool_call_id
kind
storage_path
content_type
size_bytes
checksum
created_at
retention_policy
```

보존 기간, 용량 제한, 압축과 삭제 정책은 후속 결정이다.

## Command Policy와 Owner Grant

이 플랫폼은 AI가 작동할 안전한 실행 환경을 만드는 것이며 AI가 아무 일도 못 하게 만드는 제품이 아니다. Command Policy는 AI의 일반 능력을 차단하는 장치가 아니라 사용자나 조직이 원하는 실행 환경 경계를 표현하는 설정이다.

Personal Mode에서는 하드코딩된 세밀한 명령 allowlist를 기본 설계로 두지 않는다. 수정 가능 경로, 프로젝트 외부 접근, 관리자 권한, remote push, 배포, 외부 전송과 비밀정보 접근은 Owner Grant, Project Settings와 Autonomy Profile로 허용하거나 금지한다. 기본값은 보수적으로 두되 사용자가 자신이 가진 권한 안에서 더 넓은 범위를 명시할 수 있다.

Owner는 자신의 권한을 확대할 수 없고 [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]의 R4 자동 실행 금지는 모든 프로필에서 유지된다. 팀과 Enterprise에서는 중앙 정책, 세밀한 allow/deny rule, 감사, 보존, 승인과 실행 환경 제한을 확장할 수 있으며 상세 설계는 후속 ADR로 남긴다.

## External CLI Worker Contract

External CLI Worker 실행에도 플랫폼의 엄격한 통제가 적용된다.

- External CLI Worker actions must go through Tool/Action policy.
- `external_cli.run_antigravity_experimental` is an alpha action.
- Process Runner stdin DEVNULL is part of background command policy. 백그라운드 프로세스는 대화형 프롬프트 대기로 인한 무한 대기(hang)를 방지해야 한다.
- External CLI request schemas must not accept executable/args/free-form prompt from HTTP request unless a future ADR explicitly allows it. HTTP API를 통한 임의의 명령어 주입은 차단된다.
- Active WorkerRun guard is required for external CLI runs. Worker는 할당된 활성 상태의 WorkerRun(lease) 범위에서만 실행될 수 있다.
- Environment allowlist and redaction are required. 자격 증명 누출을 막기 위해 환경 변수는 allowlist 기반으로 전달하고 민감 정보는 가려야 한다.

## 개인 모드 merge 승인

Personal Mode에서 squash merge 승인 필요 여부를 모든 설정에 대해 고정하지 않는다.

- 기본 `Allow Local Work`에서는 작업 브랜치 변경과 commit을 자동 허용하고 main/default branch squash merge에는 사용자 승인이 필요하다.
- 사용자가 더 높은 권한을 부여하면 특정 Project, repository와 risk 범위 안에서 자동 squash merge를 허용할 수 있다.
- Owner는 Approval Policy와 Owner Grant를 우회할 수 없다.
- R4 행동은 자동 실행할 수 없다.

팀 모드 merge 승인, Approval Group, Merge Coordinator와 팀 Memory 구조는 이 ADR에서 확정하지 않는다.

## v1 참고 구현 반영

기존 v1 구현 저장소 `powerpunch080403/ai-game-company-server`는 참고 자료일 뿐 새 제품 기능의 결정 원본이 아니다.

- Worker lease/claim 개념은 Personal Mode MVP에 포함하되 ADR-0009의 Task Attempt와 Worker Run에 맞게 재설계한다.
- Worker package/report 개념은 참고하되 상세 형식을 그대로 채택하지 않는다.
- Golden Path는 지금 그대로 채택하지 않고 구현이 연결된 뒤 end-to-end 검증 시나리오로 다시 설계한다.
- Discord 결합 구조, 동기식 Owner CLI 실행, v1 DB·역할·권한·서버 구조는 복사하지 않는다.
- v1과 이 ADR이 충돌하면 이 ADR과 현재 도메인 모델을 우선한다.

## 개인 모드 MVP 범위

포함한다.

- Desktop-ready Web UI
- 초기 접근 방식인 browser-accessible Web UI
- 초기 local transport인 localhost HTTP API
- Internal Tool Contract와 Tool Registry
- Tool Call Envelope와 Tool Boundary
- Local Policy Engine과 Approval Interruption
- Worker lease/claim
- `artifact_refs`
- Owner Grant와 Autonomy Profile 기반 command policy
- squash merge 승인 필요 여부의 정책 기반 평가

제외하거나 후순위로 둔다.

- Desktop App Shell 패키징과 local IPC
- 앱 자동 업데이트와 OS keychain 연동
- 최종 Tool 전체 목록 고정
- Worker package/report 상세 형식
- 팀 Approval Group 상태 머신과 Merge Coordinator 상세
- 팀 Memory 구조와 Enterprise command policy 상세
- Golden Path 최종 테스트 시나리오
- artifact retention 고급 정책
- 외부 검색, 이미지 생성 등 일반 AI 기능용 서버 Tool 제공

## 팀 모드 확장 방향

동일한 Tool Contract, Tool Call Envelope, ArtifactRef와 Approval Interruption을 팀 모드에서도 재사용한다. 모든 로컬 Tool Call이 매번 Central Authority의 허락을 받는 구조로 고정하지 않는다. Personal Node의 로컬 작업은 로컬 Tool Contract와 Local Policy Engine이 처리할 수 있다.

팀 공식 Approval Request 생성, Approval Group 결정 기록, 중앙 Merge Queue 등록과 공식 merge 기록은 Central Authority API를 거쳐 Central PostgreSQL에 기록한다. Personal Node SQLite는 로컬 Agent Run, Tool Call, Worker Run과 Worktree 작업 기록을 소유하며 중앙 상태의 동등한 writer가 아니다.

```text
Personal Node local test run
→ Personal Node SQLite에 Tool Call과 Worker Run 기록

Personal Node가 팀 공식 merge request 제출
→ Personal Node SQLite에 제출 기록
→ Central Authority HTTP API 호출
→ Central PostgreSQL에 공식 Approval Request / Merge Queue 기록
```

로컬과 중앙 기록은 `correlation_id`, `central_request_id`, `agent_run_id`, `task_attempt_id`, `change_package_id` 같은 연결 식별자로 추적한다. Approval Group, Merge Coordinator, 중앙 Merge Queue의 상세 상태 머신, 팀 Memory와 중앙 정책 캐시·위임 모델은 후속 ADR에서 결정한다.

## 장점

- UI 구현과 서버 업무 규칙을 분리해 Desktop App으로 확장할 수 있다.
- Tool Contract를 HTTP, IPC, 내부 호출과 팀 API에서 재사용할 수 있다.
- Tool Boundary와 Policy Engine의 책임이 분리된다.
- 공통 Envelope와 idempotency로 감사와 재시작 복구가 일관된다.
- Worker 중복 실행과 대형 산출물 저장 문제를 명시적으로 다룬다.

## 단점

- Tool Registry, Envelope, 정책 평가와 lease 저장소가 초기 복잡도를 높인다.
- 내부 호출과 HTTP 호출이 같은 application service 규칙을 지키도록 테스트해야 한다.
- Desktop App Shell과 IPC가 후순위라 초기 배포 경험은 여전히 브라우저 중심이다.
- 최종 Tool schema, lease timeout과 artifact 보존 정책은 별도 설계가 필요하다.

## 후속 과제

- Desktop App Shell 기술 선택: Tauri, Electron 또는 기타
- local IPC 사용 여부와 방식
- 앱 자동 업데이트와 OS keychain 연동 방식
- HTTP API 인증, 세션과 토큰 만료 정책
- 최종 Tool 목록과 각 input/output schema
- Tool version 호환성과 migration 정책
- Worker lease timeout, heartbeat와 reclaim 규칙
- Worker package/report 상세 형식
- artifact retention, compression과 deletion policy
- Golden Path end-to-end 테스트 시나리오
- 팀 Approval Group과 Merge Coordinator의 상태 머신
- 팀 Memory와 Central Authority 정책 캐시·위임 모델
- Enterprise command policy 상세

## 관련 문서

- [[01 Product/Personal Mode MVP]]
- [[02 Architecture/System Context]]
- [[02 Architecture/Data Ownership]]
- [[03 Domain Model/Domain Model]]
- [[05 Database/Database Strategy]]
- [[09 Roadmap/Personal Mode MVP Roadmap]]
- [[10 Open Questions/Open Questions]]

