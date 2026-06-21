---
type: adr
status: accepted
date: 2026-06-22
scope:
  - personal-mode
  - worker
  - test-runner
  - remote-node
  - artifact
  - tailscale
  - tool-contract
  - mvp
---

# ADR-0012: Remote Test Runner Worker Capability

## 배경

[[07 ADR/ADR-0005 Personal and Team Runtime Topology]]는 실행 환경과 중앙 Authority의 경계를 정했고, [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]는 Owner와 Worker 실행 생명주기를 분리했다. [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]는 Tool Call과 위험 기반 승인을 정했다. [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]와 [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]는 Personal Mode의 Worktree, Task Attempt와 Worker 실행을 정했다. [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]는 Tool Registry, Worker lease/claim과 Artifact Reference를 도입했다. [[07 ADR/ADR-0011 Personal Runtime, Account, Device, and Session Model]]는 Tailscale을 앱 접속 기본 전제에서 제거하고 Remote Test Runner 상세를 이 ADR로 분리했다.

이 ADR은 Personal Mode MVP에 좁은 범위의 Remote Test Runner를 포함하고 Owner, Worker와 원격 실행 환경의 계약을 정의한다.

## 문제

테스트가 다른 OS, 장치나 실행 환경을 요구할 수 있지만 Remote Test Runner를 독립 Worker나 Owner Tool로 취급하면 다음 문제가 생긴다.

- Owner가 실행 환경을 직접 조작해 Task와 Worker 경계를 우회한다.
- Test Runner가 판단, 코드 수정과 결과 확정을 담당해 Worker와 Identity가 혼합된다.
- 원격 checkout이 소스 원본이나 공식 branch처럼 취급될 수 있다.
- 로그와 대형 산출물이 메시지 본문에 섞여 복구와 감사를 어렵게 한다.
- 등록되지 않은 Node, repository와 명령이 실행될 수 있다.
- 연결 중단, 중복 dispatch와 artifact upload 실패를 추적하기 어렵다.

## 결정

- Remote Test Runner는 Owner Tool이나 독립 AI Worker가 아니라 Worker가 Task Attempt 중 사용하는 Worker Capability다.
- Owner는 테스트 Task를 만들고 Worker Report를 검토하지만 Test Runner를 직접 호출하지 않는다.
- Worker는 Task Attempt를 claim한 상태에서만 등록된 Test Runner를 호출한다.
- MVP는 Git clone/fetch/checkout 기반 test, build와 lint 실행만 지원한다.
- Remote Test Runner는 코드를 수정하거나 commit, merge, 배포하지 않는다.
- Test Runner 산출물은 Main App Artifact Store에 업로드하고 SQLite에는 `artifact_ref`를 기록한다.
- Worker가 전체 로그와 artifact를 분석해 Worker Report를 작성한다. Owner와 UI는 필요할 때 원본 artifact를 열람할 수 있다.
- Test Runner Node와 Test Run은 명시적인 상태, lease 또는 run claim과 heartbeat를 가진다.
- Tailscale은 선택적 사설 네트워크 후보다. 앱 접속 수단이나 필수 dependency가 아니다.

### 기존 ADR과의 관계

- ADR-0009의 Task, Task Attempt, Worker 실행과 repository scope를 유지한다.
- ADR-0010의 Worker claim, Tool Registry, Tool Call Envelope, Policy Engine과 ArtifactRef를 재사용한다.
- ADR-0011의 “Tailscale은 앱 접속 기본 수단이 아니다”라는 결정을 유지한다.
- 이 ADR은 ADR-0011에서 후속으로 남긴 Remote Test Runner 등록, 실행, artifact와 상태 경계를 구체화한다.

## 용어 정의

```text
Owner
= 사용자의 의도를 해석하고 Task를 만들며 Worker 결과를 검토하는 주체

Worker
= Task Attempt를 claim하고 실제 작업을 수행하는 실행 주체

Remote Test Runner
= Worker가 test/build/lint 실행을 위해 원격으로 사용하는 실행 환경

Test Runner Tool / Capability
= Worker가 Remote Test Runner를 호출하고 결과를 수집하기 위한 기능

Artifact Store
= 테스트 로그, screenshot, report와 파일 산출물을 저장하는 Main App 저장소

artifact_ref
= Artifact Store에 저장된 원본 파일을 가리키는 참조
```

Remote Test Runner는 판단 주체가 아니라 Worker가 사용하는 실행 환경이다. 독립 AI Worker가 아니며 Owner가 직접 조종하는 Tool도 아니다.

## Owner, Worker, Test Runner의 역할 분리

```text
User request
→ Owner creates or updates Task
→ Worker claims Task Attempt
→ Worker uses Test Runner Capability
→ Remote Test Runner executes test/build/lint
→ Artifacts uploaded to Main App Artifact Store
→ Worker analyzes logs and artifacts
→ Worker submits Worker Report with artifact_refs
→ Owner reviews Worker Report
→ Owner reads original artifacts if needed
→ Owner decides next action
```

Owner는 테스트가 필요하면 테스트 Task를 만들거나 기존 Task의 success criteria와 실행 범위에 테스트를 포함한다. Worker가 Task Attempt를 claim하고 어떤 test target과 명령을 실행할지 판단한다. Test Runner는 지정된 명령을 실행하고 증거를 반환한다.

Worker는 로그와 artifact를 분석해 Report를 만든다. Owner는 Report를 검토하고 필요하면 artifact Tool로 원본을 읽은 뒤 완료, 수정 Task, 재시도 Task 또는 사용자 승인 요청을 결정한다.

## Remote Test Runner를 Worker Capability로 보는 이유

Test Runner는 Worker Tool 또는 Worker Capability다. Tool Registry는 Owner Tool과 Worker Capability의 허용 caller와 scope를 구분할 수 있어야 한다.

`test_runner.run` 같은 호출에는 `caller_type = worker` 또는 이에 준하는 정보가 Tool Call Envelope에 남는다. Worker 호출도 Tool Boundary, Policy Engine, 권한 검사와 감사 기록을 거친다. Worker는 Task Attempt 없이 Runner를 호출할 수 없고 Task, Project, Repository, Owner Grant와 Project Settings가 허용한 범위 안에서만 사용할 수 있다.

초기 Capability 후보는 다음과 같으며 최종 고정 목록은 아니다.

```text
test_runner.register
test_runner.get_status
test_runner.run
test_runner.cancel
test_runner.collect_artifacts
test_runner.upload_artifact
test_runner.read_log
```

## 원격 연결 모델

논리적 실행 방향은 Worker에서 Remote Test Runner로 향한다.

```text
Worker
→ Test Runner Capability
→ Registered Remote Test Runner Agent
→ test/build/lint process
```

물리 transport는 local HTTP, agent channel, WebSocket, SSH 또는 Tailscale IP 접근으로 확장할 수 있다. MVP 우선 후보는 Tailscale 또는 같은 사설 네트워크 위에서 Main App의 Local Runtime이 등록된 Test Runner Agent에 접속하는 방식이다.

Remote Test Runner는 작업을 스스로 정의하거나 임의 Queue에서 가져가지 않는다. Worker가 Task Attempt 범위에서 test target을 결정하고 등록된 명령을 요청한다.

NAT traversal, relay server와 public internet exposure는 MVP에서 제외한다. Runner가 outbound polling으로 Main App에 연결하는 방식은 후속 대안으로 남긴다. 정확한 network protocol은 후속 설계다.

## Tailscale의 위치

Tailscale은 앱 접속 수단이 아니라 Remote Test Runner에 안전하게 도달하기 위한 선택적 network option이다. 사용자가 여러 기기에서 앱을 쓰는 목표는 [[07 ADR/ADR-0011 Personal Runtime, Account, Device, and Session Model]]의 중앙 account와 동기화 방향을 따른다.

Tailscale을 사용하지 않아도 같은 LAN, SSH tunnel 또는 향후 relay 방식으로 대체할 수 있어야 한다. MVP는 Tailscale을 강제하지 않고 지원 가능한 사설 네트워크 구성 후보로 둔다. 인터넷 직접 공개는 기본값이 아니다.

## Test Runner 등록과 신뢰

등록된 Test Runner Node만 사용할 수 있다. MVP는 pairing code 또는 등록 token을 통해 Node를 등록하고 `test_runner_node_id`를 부여한다. 중앙 account 기반 등록은 후순위다.

Test Runner Node는 사용자 Device, Human, Worker Identity와 구분되는 실행 Node다. ADR-0011의 Device 모델과 nullable 관계를 가질 수 있지만 사용자 Session과 같은 것으로 취급하지 않는다.

후보 속성:

```text
test_runner_node_id
device_id nullable
display_name
network_address
network_kind: tailscale | lan | ssh_tunnel | relay_candidate | unknown
capabilities
allowed_project_ids
allowed_repository_ids
status
last_seen_at
registered_at
revoked_at
```

Node는 Project, Repository와 Capability별 허용 범위를 가진다. 등록됐다는 사실만으로 모든 Task를 실행할 수 없고 Worker가 유효한 Task Attempt 없이 호출할 수도 없다.

## 테스트 실행 흐름

1. 사용자가 테스트나 빌드를 요청한다.
2. Owner가 테스트 Task를 만들거나 기존 Task에 테스트 단계를 포함한다.
3. Worker가 Task Attempt를 claim한다.
4. Worker가 repository 상태, branch, commit과 test target을 확인한다.
5. Worker가 허용되고 사용 가능한 Remote Test Runner를 선택한다.
6. Worker가 Test Runner Capability를 호출한다.
7. Runner가 지정된 repository, branch와 commit을 준비한다.
8. Runner가 test, build 또는 lint 명령을 실행한다.
9. Runner가 로그, screenshot, report와 파일 산출물을 Main App Artifact Store에 업로드한다.
10. Local Runtime SQLite에 `artifact_ref`가 저장된다.
11. Worker가 artifact를 읽고 분석한다.
12. Worker가 Worker Report를 작성한다.
13. Owner가 Report와 필요 시 원본 artifact를 검토한다.
14. Owner가 완료, 재시도, 수정 Task 또는 사용자 승인 요청을 결정한다.

## 코드와 repository 처리

MVP Remote Test Runner는 Git checkout 기반으로 테스트한다. Main App이 임의 파일 묶음을 기본 전송하지 않는다.

Worker는 `repository_id`, remote URL, branch name, commit SHA와 test target을 명시한다. Runner는 repository를 clone 또는 fetch하고 지정된 commit이나 branch를 checkout한다. 가능하면 immutable commit SHA를 사용해 재현성을 높인다.

Runner의 checkout은 임시 실행 복사본이며 소스 원본은 Git repository다. Runner는 테스트 대상 코드를 수정하거나 commit을 만들지 않고 main/default branch와 remote를 변경하지 않는다.

대형 파일 package 전송, Git repository가 없는 Project와 binary asset sync는 후속 과제다.

## Artifact 업로드와 Owner 열람

Runner가 만든 파일과 전체 로그를 메시지 본문으로 Owner에게 직접 전달하지 않는다. Runner는 산출물을 Main App의 Artifact Store에 업로드하고 Local Runtime SQLite는 `artifact_ref`를 기록한다.

```text
Remote Test Runner output
→ Main App Artifact Store
→ artifact_ref in Local Runtime SQLite
→ Worker analysis and Worker Report
→ Owner/UI review
```

Worker는 `artifact_ref`로 로그와 파일을 읽고 분석한다. Worker Report에는 요약, 실패 원인 추정, 재시도 내역과 관련 `artifact_ref`가 포함된다. Owner는 Report를 우선 검토하고 필요하면 artifact Tool로 원본을 열람한다. UI도 `artifact_ref`를 통해 로그, screenshot과 report를 표시할 수 있어야 한다.

`artifact_ref`는 Owner의 접근을 막는 축약본이 아니다. 대형 산출물을 안전하게 저장·추적하고 Owner와 UI가 필요할 때 원본을 열람하기 위한 참조다.

Owner가 사용할 수 있는 artifact Tool 초기 후보:

```text
artifact.get_metadata
artifact.read_text
artifact.preview_image
artifact.download_ref
artifact.list_for_task_attempt
```

Artifact kind 후보:

```text
test_log
build_log
lint_log
screenshot
screen_recording
coverage_report
crash_dump
generated_report
test_result_json
binary_artifact
```

## Worker Report와 재시도

Worker는 원본 로그를 그대로 넘기는 데서 끝나지 않고 실행 환경, test target, branch/commit, 명령, 성공·실패, 핵심 로그와 실패 원인을 분석한다.

Worker Report 후보 속성:

```text
worker_report_id
task_attempt_id
test_run_id
repository_id
commit_sha
test_target
environment
command_summary
status
summary
failure_hypothesis
retry_count
retry_changes
artifact_refs
created_at
```

실패하면 Worker는 Task 범위와 Tool Policy가 허용한 설정만 바꿔 재시도할 수 있다. 예를 들어 timeout 증가, headless/headed 전환, 특정 test 재실행과 log level 증가를 사용할 수 있다. 재시도는 새 Test Run 또는 `retry_of_test_run_id`로 기록한다.

재시도 횟수와 변경 가능한 설정은 Project Settings 또는 Tool Policy로 제한한다. Worker가 테스트 실패를 근거로 코드를 수정할 수 있는지는 원래 Task의 `write_scope`에 달려 있다. Remote Test Runner 자체는 어떤 경우에도 코드를 수정하지 않는다.

## 권한과 금지 사항

금지한다.

```text
Remote Test Runner가 코드 수정
Remote Test Runner가 commit 생성
Remote Test Runner가 merge 수행
Remote Test Runner가 main/default branch 변경
Remote Test Runner가 Task 없이 임의 실행
Remote Test Runner가 등록되지 않은 repository 접근
Remote Test Runner가 허용되지 않은 Project/Repository 테스트
Remote Test Runner가 Owner를 우회해 결과 확정
Remote Test Runner가 사용자 승인 없이 배포
Remote Test Runner가 외부 인터넷에 artifact를 임의 전송
```

허용한다.

```text
등록된 repository clone/fetch/checkout
허용된 test/build/lint 명령 실행
테스트 로그 수집
screenshot 또는 screen recording 수집
crash dump와 coverage report 수집
Main App Artifact Store로 artifact 업로드
heartbeat와 status 보고
Worker가 요청한 취소 처리
```

모든 호출은 Tool Boundary와 Policy Engine을 거친다. caller는 Worker여야 하고 유효한 Task Attempt claim을 가져야 한다. 대상 Project, Repository와 test target은 Task scope와 Node 등록 범위 안에 있어야 한다. 위험한 명령, 배포와 외부 전송은 이 Capability 범위 밖이며 별도 Tool과 승인 정책 없이는 실행할 수 없다.

## 상태와 lease/heartbeat

Remote Test Runner Node는 상태와 heartbeat를 가진다. Main App은 `last_seen_at`과 현재 실행 상태를 저장한다.

Node 상태 후보:

```text
registered
available
busy
offline
unhealthy
revoked
unknown
```

Test Run은 lease 또는 run claim을 가진다. 실행 중 heartbeat가 끊기면 즉시 성공이나 실패로 단정하지 않고 timeout 또는 `lost`로 전환할 수 있다. Worker가 상태를 확인해 재시도, 취소 또는 실패 처리를 결정한다.

Test Run 상태 후보:

```text
created
queued
dispatching
running
uploading_artifacts
analyzing
succeeded
failed
cancelled
timed_out
lost
```

Test Run 후보 속성:

```text
test_run_id
task_attempt_id
worker_run_id
test_runner_node_id
repository_id
commit_sha
branch_name
test_target
command
status
started_at
completed_at
timeout_at
retry_of_test_run_id
artifact_refs
error_code
error_message
```

정확한 lease timeout, heartbeat 주기와 reclaim 정책은 후속 설정이다.

## 개인 모드 MVP 범위

포함한다.

- Remote Test Runner를 Worker Capability로 모델링
- Owner가 직접 Runner를 호출하지 않는 구조
- Worker가 Task Attempt 범위에서 Runner 사용
- 등록된 Remote Test Runner Node
- Tailscale 또는 같은 사설 네트워크 연결 후보
- pairing code 또는 등록 token 기반 Node 등록
- Git clone/fetch/checkout 기반 test/build/lint
- Main App Artifact Store 업로드와 `artifact_refs`
- Worker Report와 제한된 테스트 재시도
- Node heartbeat/status와 Test Run 상태 기록
- 취소와 timeout

제외하거나 후순위로 둔다.

- Remote Test Runner의 코드 수정, commit과 merge
- Remote AI Worker Node
- 중앙 account 기반 Test Runner 등록
- relay server와 public internet exposure
- 자동 scaling과 팀 공유 Worker Pool
- GPU/OS별 고급 scheduling
- 파일 package 기반 코드 전송과 binary asset sync
- 원격 배포 실행

## 팀 모드 확장 방향

Team Mode에서는 Central Authority가 공유 Test Runner Pool, 조직 정책, 승인과 사용량 정책을 소유할 수 있다. Personal Node의 로컬 Test Runner 기록은 중앙 공식 기록의 동등한 writer가 아니다.

팀 공유 Runner는 Central Authority의 등록, 권한과 감사 정책을 따라야 한다. 공식 CI, Merge Queue와 Approval Group에 연결되는 Test Run은 후속 ADR에서 설계한다. Enterprise에는 세밀한 command policy, artifact retention, network policy, runner isolation과 secret handling이 추가로 필요하다.

## v1 참고 구현 반영

기존 v1 저장소 `powerpunch080403/ai-game-company-server`는 참고 자료일 뿐 새 제품 기능의 결정 원본이 아니다.

- v1 Worker lease/claim은 Test Run lease와 heartbeat의 참고 개념으로만 사용한다.
- v1 test runner contract 아이디어는 Worker Capability와 `artifact_ref` 모델에 맞게 다시 작성한다.
- Discord 결합, 동기식 Owner CLI 실행과 v1 DB·역할·권한·서버 구조는 복사하지 않는다.
- v1 Golden Path는 그대로 채택하지 않고 구현이 연결된 뒤 end-to-end 검증 기준으로 다시 설계한다.
- v1과 이 ADR이 충돌하면 이 ADR과 현재 도메인 모델을 우선한다.

## 장점

- Owner, Worker와 원격 실행 환경의 역할이 분리된다.
- 다른 OS와 장치에서 재현 가능한 test/build/lint를 수행할 수 있다.
- 원본 artifact를 보존하면서 Worker가 결과를 분석해 Owner의 검토 부담을 줄인다.
- Git commit SHA와 Test Run 기록으로 실행 증거를 추적할 수 있다.
- Tailscale을 강제하지 않고 안전한 사설 네트워크 후보로 사용할 수 있다.

## 단점

- Runner Agent, 등록, heartbeat, 취소와 artifact upload 구현이 필요하다.
- Git clone/fetch 비용과 원격 환경 차이를 관리해야 한다.
- 사설 네트워크가 없는 환경의 relay는 MVP에서 지원하지 않는다.
- secret 전달, sandboxing과 command policy의 세부가 아직 미정이다.

## 후속 과제

- Remote Test Runner Agent 구현 방식
- 정확한 network protocol과 Tailscale 자동 감지 여부
- LAN, SSH tunnel과 relay 지원 순서
- command allow/deny policy와 sandboxing
- secret과 environment variable 전달
- artifact upload protocol과 재개
- screenshot와 screen recording capture
- binary artifact retention
- Node credential rotation
- lease timeout, heartbeat와 reclaim 값
- Remote AI Worker Node 지원 여부
- 팀 공유 Test Runner Pool과 Enterprise isolation

## 관련 문서

- [[01 Product/Personal Mode MVP]]
- [[02 Architecture/System Context]]
- [[02 Architecture/Data Ownership]]
- [[03 Domain Model/Domain Model]]
- [[05 Database/Database Strategy]]
- [[09 Roadmap/Personal Mode MVP Roadmap]]
- [[10 Open Questions/Open Questions]]

