---
type: adr
status: accepted
date: 2026-06-22
scope:
  - personal-mode
  - local-runtime
  - desktop-app
  - account
  - device
  - session
  - pairing
  - authentication
  - sync
  - tailscale
---

# ADR-0011: Personal Runtime, Account, Device와 Session 모델

## 배경

[[07 ADR/ADR-0005 Personal and Team Runtime Topology]]는 개인 모드와 팀 모드의 실행 위치를 분리했고, [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]는 재개 가능한 Owner Runtime을 정의했다. [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]는 장치와 Session 권한도 포함할 수 있는 승인 경계를 정했다. [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]는 Primary Personal Server와 브라우저 기반 첫 UI를 채택했으며, [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]는 로컬 SQLite의 핵심 상태를 정했다. [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]는 Desktop-ready Web UI, 장기 Desktop App 목표와 Local Transport를 구체화했다.

이 ADR은 기존의 “Primary Personal Server”를 별도 서버 제품이 아니라 앱이 관리하는 Local Runtime으로 정확히 설명하고, local user, future account, Device, Session과 pairing의 관계를 정한다.

## 문제

기존 표현만으로는 다음 오해가 생길 수 있다.

- 사용자가 서버 주소와 네트워크를 직접 관리해야 하는 서버 제품처럼 보인다.
- Tailscale 설치가 Personal Mode 앱 사용의 필수 조건처럼 읽힌다.
- local user와 미래 중앙 account가 하나의 Identity로 섞인다.
- Device와 Session을 구분하지 않아 장치 폐기, Session 만료와 재인증을 독립적으로 다루기 어렵다.
- 초기 Web UI pairing과 장기 Desktop App 인증 저장 방식을 성급하게 동일시할 수 있다.
- 다중 기기라는 장기 목표가 MVP 중앙 동기화 구현으로 오해될 수 있다.
- Remote Test Runner의 네트워크·권한 모델이 사용자 Session 설계와 섞일 수 있다.

## 결정

- Personal Server는 사용자가 별도로 접속·운영하는 서버 제품이 아니라 Desktop App 또는 초기 Web UI가 관리하는 Local Runtime이다.
- 장기 제품 UX는 Desktop App이며, 내부적으로 Desktop App이 Local Runtime을 시작하고 관리한다.
- 초기 구현은 Desktop App Shell에서 재사용할 수 있는 browser-accessible Desktop-ready Web UI다.
- Personal Mode MVP는 첫 설치 시 local user 한 명을 자동 생성하고 중앙 account 연결은 nullable 관계로 남긴다.
- Device와 Session을 분리하고 Web UI는 짧은 수명의 1회용 pairing code로 Local Runtime에 연결한다.
- Session은 opaque random token을 사용하고 SQLite에는 token hash만 저장한다. JWT를 기본값으로 채택하지 않는다.
- MVP에 장치·Session 목록과 폐기, pairing code 재발급 절차를 포함한다.
- 다중 기기 계정 동기화는 장기 목표지만 MVP에서는 각 장치가 독립 Local Runtime을 가진다.
- Tailscale은 앱 접속의 기본 전제가 아니라 원격 실행 Node와 고급 네트워크 구성을 위한 선택지다.
- Remote Test Runner 상세 설계는 ADR-0012로 분리한다.

### 기존 ADR과의 관계

- ADR-0005의 Personal Node와 중앙 Authority 소유권 경계는 유지한다.
- ADR-0008의 브라우저 우선 초기 구현, Windows·Linux Runtime과 SQLite 기반 복구 결정은 유지한다.
- ADR-0008의 “Tailscale을 Personal Mode 원격 접속의 기본 전제로 둔다”는 부분은 이 ADR이 명시적으로 대체한다.
- ADR-0008의 Primary Personal Server 구성은 내부 Local Runtime 배치로 재해석하며 별도 서버 제품 UX로 사용하지 않는다.
- ADR-0010의 Desktop-ready Web UI, 장기 Desktop App 목표와 Local Transport 결정은 유지하고 App-managed Local Runtime 관계로 구체화한다.

## Personal Runtime의 정의

```text
Personal Server는 사용자가 직접 접속하는 별도 서버 제품이 아니라,
Desktop App 또는 초기 Web UI가 관리하는 Local Runtime이다.
```

사용자에게는 “서버 주소에 접속한다”가 아니라 “앱을 실행해 AI 개발 플랫폼을 사용한다”는 경험을 제공한다. Local Runtime은 제품의 내부 실행 기반이다. 기존 문서와 내부 구현에서 `Primary Personal Server`라는 이름을 유지할 수 있지만, 제품 UX와 사용자 문서에서는 `앱이 관리하는 Local Runtime`으로 설명한다.

```text
UI Shell
→ Local Transport
→ App-managed Local Runtime
   ├─ Local Control Plane
   ├─ Owner Runtime
   ├─ Agent Run Engine
   ├─ Worker Supervisor
   ├─ SQLite
   ├─ Git Worktree
   └─ Artifact Store
```

## Desktop App과 Background Local Runtime

```text
제품 UX는 Desktop App이다.
내부적으로는 Desktop App이 Local Runtime을 시작하고 관리한다.
장기적으로 Local Runtime은 Background Service로 분리될 수 있다.
```

Desktop App은 UI Shell과 Local Runtime 관리자를 포함하는 장기 제품 형태다. 앱 창을 닫아도 승인 대기 전까지 작업이 계속될 수 있는 구조를 목표로 한다. 이를 위해 Local Runtime은 장기적으로 Background Local Runtime 또는 Background Service로 분리할 수 있다.

Background Service 설치 방식은 이 ADR에서 확정하지 않는다. Linux systemd, Windows Service와 사용자 Session 실행은 모두 후속 후보다.

## 초기 Web UI와 Desktop App 전환

```text
초기 구현 형태: Browser-accessible Desktop-ready Web UI
장기 제품 형태: Desktop App Shell
공통 원칙: UI는 Local Runtime을 직접 우회하지 않고 Local Transport를 통해 호출한다.
```

MVP는 개발 속도를 위해 브라우저 접근 가능한 Web UI로 시작하지만 이를 최종 제품 형태로 고정하지 않는다. Web UI의 컴포넌트와 application 흐름은 향후 Desktop App Shell 안에서 재사용할 수 있어야 한다.

[[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]에 따라 HTTP API는 브라우저 전용이 아니라 Local Transport 후보 중 하나다. Desktop App Shell은 local HTTP API 또는 후속 IPC로 Local Runtime과 통신할 수 있다. Desktop App 패키징, local IPC, 자동 업데이트와 OS keychain 연동은 후속 설계다.

## Local User와 Future Account

Personal Mode MVP는 첫 설치 시 local user 한 명을 자동 생성한다. 회원가입이나 중앙 로그인이 없어도 프로젝트 소유권, 권한 위임, 장치와 Session을 특정 Human 사용자에게 연결할 수 있어야 한다.

local user와 future central account는 서로 다른 Identity다. local user가 중앙 account에 연결될 수 있지만 둘을 하나의 ID로 혼합하지 않는다.

후보 관계:

```text
LocalUser
├─ local_user_id
├─ account_id nullable
├─ account_link_status: local_only | linked | unlinked | revoked
├─ created_at
└─ updated_at
```

MVP는 중앙 account 로그인, 회원가입과 동기화 서버를 구현하지 않는다. `account_id` 또는 `external_account_id`는 nullable 연결 정보로 둔다. 중앙 account는 향후 다중 기기 동기화, 라이선스·구독과 팀 멤버십의 기반이 될 수 있다.

## Device와 Session 모델

```text
Device = 신뢰되거나 등록된 앱, 브라우저 또는 Node 단위
Session = 특정 Device에서 현재 인증된 접속 상태
```

Session만 두지 않고 Device를 별도 엔티티로 둔다. Device는 장치 이름, type, 계정 연결, 마지막 접속과 폐기를 추적한다. Session은 로그인 상태, 만료, 폐기와 재인증을 관리한다.

Device type 후보:

```text
desktop_app
web_ui
local_runtime
worker_node
test_runner_node
unknown
```

`worker_node`와 `test_runner_node`는 확장 가능한 type 후보일 뿐 상세 모델은 이 ADR에서 확정하지 않는다. Remote Test Runner 관련 상세는 ADR-0012에서 다룬다.

Session 후보 속성:

```text
session_id
device_id
local_user_id
account_id nullable
token_hash
created_at
last_seen_at
idle_expires_at
absolute_expires_at
revoked_at
```

사용자는 설정에서 등록 Device와 활성·폐기된 Session을 볼 수 있어야 한다. 특정 Device 폐기, 특정 Session 폐기와 모든 Session 로그아웃을 지원한다. device name, device type, 마지막 접속과 대략적인 접속 출처를 표시한다. IP 주소, Tailscale 이름과 OS 정보의 정확한 수집·보존 정책은 후속으로 조정한다.

## Pairing Code와 Web UI 접속

초기 Web UI는 Local Runtime이 생성한 pairing code로 연결한다.

1. Local Runtime이 짧은 수명의 pairing code를 발급한다.
2. 사용자가 Web UI에 code를 입력한다.
3. Local Runtime이 code를 검증하고 Device와 Session을 발급한다.
4. pairing code는 1회 사용 후 폐기한다.

MVP 권장값:

```text
pairing code expiry: 10분
usage: 1회
storage: code 원문이 아닌 hash
```

pairing code는 원격 접속 기능이 아니라 Web UI를 Local Runtime에 연결하는 인증 절차다.

Session token은 폐기 가능한 opaque random token을 사용하고 SQLite에는 token hash만 저장한다. JWT를 기본값으로 사용하지 않는다. Web UI는 HttpOnly, SameSite cookie 사용을 권장하며 localStorage에 민감 token을 저장하지 않는다.

MVP Session 만료 권장값:

```text
idle timeout: 30일
absolute timeout: 90일
```

정확한 기간은 구현 검증 결과에 따라 후속 조정할 수 있다.

모든 Session을 잃었거나 새 Web UI를 연결해야 하면 Local Runtime에서 새 pairing code를 재발급할 수 있어야 한다. MVP는 CLI 또는 Local Runtime console에 공식 재발급 절차를 제공한다. 명령 형태 후보는 `aidev auth pairing-code`이며 정확한 CLI 명령은 구현 시 확정한다. SQLite 직접 수정은 공식 복구 절차가 아니다. Desktop App Shell이 생기면 앱 내부 복구 UI로 확장할 수 있다.

## 여러 기기와 동기화 방향

```text
장기 목표는 account 기반 다중 기기 동기화다.
MVP는 독립 Local Runtime으로 시작하되,
account/device 식별자를 통해 후속 동기화를 막지 않는다.
```

장기적으로 같은 account로 여러 기기에서 프로젝트, Conversation과 설정을 동기화하는 것을 목표로 한다. 그러나 MVP에서는 각 장치가 독립 Local Runtime과 SQLite를 가지며 중앙 동기화를 구현하지 않는다.

데이터 모델에는 `account_id`, `device_id`와 sync-ready metadata를 둘 수 있다. Conversation, Task, Agent Run, Artifact와 Git repository 상태의 동기화 방식, 충돌 해결과 원본 소유권은 후속 설계다.

## Tailscale의 위치

Tailscale은 Personal Mode 앱 접속의 기본 전제가 아니다. MVP 기본 사용 흐름은 Tailscale 설치나 사전 구성을 요구하지 않으며 인터넷 직접 공개도 기본값으로 하지 않는다.

```text
Tailscale은 기본 앱 접속 수단이 아니라,
원격 실행 Node와 고급 네트워크 구성을 위한 선택지다.
```

다중 기기 앱 사용이라는 장기 목표는 중앙 account와 동기화 모델로 해결한다. Tailscale은 Remote Test Runner, Worker Node와 고급 사설 네트워크 구성에 선택적으로 사용할 수 있다. 구체적인 원격 Node 연결 방식은 ADR-0012에서 다룬다.

## Remote Test Runner와의 경계

Remote Test Runner는 Personal Mode MVP에 포함할 수 있는 중요한 확장 기능이지만 account, Device와 사용자 Session 모델과는 별도 축이다. 상세 설계는 ADR-0012에서 다룬다.

ADR-0012에서 다룰 방향은 다음과 같다.

- Test Runner는 Worker Tool 또는 Worker Capability로 모델링한다.
- Owner는 Test Runner를 직접 조작하지 않고 테스트 Task를 만든다.
- Worker가 Task를 받아 Remote Test Runner를 사용한다.
- Test Runner는 판단 주체가 아니라 테스트 실행 환경이다.
- Worker가 전체 로그와 산출물을 분석하고 Worker Report를 만든다.
- 실패 시 허용된 Task 범위에서 테스트 설정을 변경해 재시도할 수 있다.
- 로그, screenshot, 파일과 report는 Main App의 Artifact Store에 업로드하고 SQLite에는 `artifact_ref`를 둔다.
- Worker Report는 `artifact_ref`를 포함한다.
- Owner는 Report를 우선 검토하고 필요하면 artifact Tool로 원본을 열람한다.
- `artifact_ref`는 접근을 제한하는 축약본이 아니라 대용량 증거를 안전하게 저장·추적하고 필요 시 열람하기 위한 참조다.

이 ADR은 Remote Test Runner의 상태 머신, 네트워크 연결, 권한, 등록·heartbeat와 artifact upload protocol을 확정하지 않는다.

## 개인 모드 MVP 범위

포함한다.

- App-managed Local Runtime 개념
- 초기 Desktop-ready Web UI와 향후 Desktop App Shell 재사용 전제
- local user 자동 생성과 nullable `account_id`
- Device와 Session 분리
- Web UI pairing code
- opaque Session token과 token hash 저장
- HttpOnly, SameSite cookie 권장
- Device·Session 목록과 폐기
- pairing code 재발급 CLI 또는 console 절차
- Tailscale을 기본 앱 접속 전제에서 제거

제외하거나 후순위로 둔다.

- 실제 Desktop App Shell 패키징
- Background Service 설치 방식
- OS keychain 연동
- 중앙 account 로그인과 회원가입
- 다중 기기 동기화
- 라이선스·구독과 팀 멤버십
- Remote Test Runner 상세 설계
- Tailscale 기반 원격 Node 연결 방식

## 팀 모드 확장 방향

Team Mode에서는 Central Authority가 중앙 account, Membership과 조직 권한의 원천이다. Personal Node의 local user는 중앙 account와 연결될 수 있지만 중앙 account Session과 로컬 앱 Session은 구분한다.

다중 기기 동기화, 팀 Membership과 중앙 승인 정책은 Central Authority와 PostgreSQL로 확장한다. Personal Node의 로컬 SQLite는 중앙 PostgreSQL의 동등한 writer가 아니다. 팀 인증, 조직 Membership, 중앙 Session과 Device 관리의 상세는 후속 ADR에서 다룬다.

## 장점

- 사용자가 서버를 운영한다는 느낌 없이 앱 중심 UX를 설명할 수 있다.
- local user, account, Device와 Session의 Identity 경계가 명확해진다.
- 초기 Web UI와 장기 Desktop App을 하나의 전환 경로로 연결한다.
- pairing과 Session 폐기를 MVP에서 복구 가능한 인증 흐름으로 제공한다.
- 다중 기기 목표를 유지하면서 MVP 동기화 범위를 제한한다.
- Tailscale과 Remote Test Runner를 기본 앱 접속에서 분리한다.

## 단점

- 초기 Web UI에서는 pairing, cookie와 Session 관리 구현이 필요하다.
- Desktop App과 Background Service의 설치·token 저장 방식은 아직 미정이다.
- nullable account 연결과 sync-ready metadata가 MVP schema를 다소 복잡하게 한다.
- 독립 Local Runtime 사이에는 MVP 단계에서 데이터가 자동 동기화되지 않는다.

## 후속 과제

- Desktop App Shell 기술 선택
- Background Service 설치 방식: Linux systemd, Windows Service 또는 사용자 Session
- OS keychain 연동 방식
- 중앙 account 로그인, 회원가입, password와 OAuth 정책
- 다중 기기 동기화와 conflict resolution
- pairing code의 정확한 표시·복구 UX
- Session token refresh 정책과 기간 검증
- Device 정보와 접속 출처의 보존 정책
- 팀 중앙 account와 로컬 Session 연결 방식
- ADR-0012 Remote Test Runner, 원격 Node 인증과 artifact 흐름 상세

## 관련 문서

- [[01 Product/Personal Mode MVP]]
- [[02 Architecture/System Context]]
- [[02 Architecture/Data Ownership]]
- [[03 Domain Model/Domain Model]]
- [[05 Database/Database Strategy]]
- [[09 Roadmap/Personal Mode MVP Roadmap]]
- [[10 Open Questions/Open Questions]]
