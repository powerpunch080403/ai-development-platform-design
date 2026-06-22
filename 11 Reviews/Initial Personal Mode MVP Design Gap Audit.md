# Initial Personal Mode MVP Design Gap Audit

## Purpose
ADR-0014에서 확정된 Owner-led Worker 구조와 엄격한 통제 정책(write_scope 강제, 사용자-Worker 직접 대화 금지, 실패 Worktree 보존 등)을 기준으로, 기존 설계 문서들의 빈틈이나 충돌을 파악한다. 이 문서는 초기 MVP 구현을 시작하기 전에 어떤 설계 항목이 확정되었고, 어떤 부분이 아직 모호한지 분류하여 ADR-0016 및 후속 설계에서 채워야 할 범위를 정의한다.

## Reviewed Documents
- `README.md`
- `00 HOME.md`
- `01 Product/Personal Mode MVP.md`
- `01 Product/Personal Mode Complete Experience.md`
- `02 Architecture/System Context.md`
- `02 Architecture/Data Ownership.md`
- `03 Domain Model/Domain Model.md`
- `05 Database/Database Strategy.md`
- `07 ADR/ADR-0007 Autonomy and Approval Risk Policy.md`
- `07 ADR/ADR-0008 Personal Mode MVP and Deployment.md`
- `07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines.md`
- `07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API.md`
- `07 ADR/ADR-0011 Personal Runtime, Account, Device, and Session Model.md`
- `07 ADR/ADR-0013 MVP Implementation Slice and Repository Strategy.md`
- `07 ADR/ADR-0014 Owner-Led AGY Worker Alpha and Safe Pilot Policy.md`
- `07 ADR/ADR-0015 Personal Mode Completion Scope and Design Roadmap.md`
- `09 Roadmap/Personal Mode MVP Roadmap.md`
- `10 Open Questions/Open Questions.md`

## Summary
전반적으로 Owner-led Worker 모델, write_scope 강제, Worker 권한 제한, 실패 worktree 보존 등의 핵심 정책은 최신 ADR과 로드맵에 일관되게 잘 반영되어 있다. 
그러나 **Owner의 작업 계획 분할(Decomposition) 방식, 실패/타임아웃 발생 시의 재시도(Retry/Resume) 구체적 행동, Owner preflight의 상세 결과 처리, Tailscale 필수 여부 관련 충돌** 등이 빈틈으로 발견되었다.

## Findings by Area

### A. Owner planning and Task creation
- **Decided**: Task instruction의 최소 필수 필드, Task 없이 Worker 실행 불가 정책. (Evidence: ADR-0009, ADR-0014)
- **Partially decided**: Owner가 Task를 만들 때 사용자 승인 없이 바로 큐잉할 수 있는지. (Evidence: Roadmap Phase D, ADR-0007)
- **Undecided**: 사용자의 넓은 요청(broad request)을 Owner가 구체적 Task로 쪼개는 내부 프롬프트와 UX. (Gap: Decomposition logic)
- **Blocker**: Owner가 넓은 요청을 어떻게 Task로 나누고 UI에 표시할 것인가 (ADR-0016 대상).

### B. write_scope and risk
- **Decided**: write_scope는 Owner가 정하며 Worker가 스스로 확장할 수 없음. (Evidence: ADR-0009, ADR-0014)
- **Undecided**: 기본 write_scope가 무엇인지, 디렉토리 단위인지 개별 파일 단위인지. (Gap: Default write_scope granularity)
- **Undecided**: protected path, secret file 등의 명시적 기준과 `allow_new_files` 기본값. (Gap: Protective scope limits)
- **Blocker**: 초기 구현에서 적용할 기본 write_scope 정책.

### C. Owner preflight
- **Decided**: Preflight는 사용자 개입 없이 Owner의 readiness check로 수행됨. (Evidence: ADR-0014)
- **Partially decided**: WorkerRun active guard는 도입되나(ADR-0010), preflight가 실패했을 때 Agent Run의 정확한 상태 전이.
- **Undecided**: Preflight 결과를 DB나 Artifact에 어떻게/어디까지 기록할 것인가. (Gap: Preflight observability)
- **Recommendation**: Preflight 실패는 Agent Run의 `failed` 또는 `waiting_for_user`로 처리되도록 구체화.

### D. Worker execution and adapters
- **Decided**: AGY Alpha는 opt-in이며 기본 MVP 테스트에선 skip됨. External CLI는 free-form prompt를 받지 않음. (Evidence: ADR-0008, ADR-0010, ADR-0014)
- **Partially decided**: WorkerRun의 세부 상태 전이(ADR-0010에 후보만 있음)와 ProcessRun의 상태 정의.
- **Non-blocker**: WorkerRun의 구체적 상태 필드는 구현 과정에서 확정해도 됨.

### E. Failure, resume, retry, cleanup
- **Decided**: 실패 worktree는 자동 삭제되지 않고 보존됨, cleanup은 명시적 요청 또는 병합 후 수행. (Evidence: ADR-0009, ADR-0014, Roadmap)
- **Partially decided**: 실패 Attempt를 이어서 작업할 때, 같은 Attempt ID를 쓸 것인가 새 Attempt를 만들 것인가. (Gap: Resume semantics)
- **Partially decided**: abandon과 cleanup의 구체적 동작 차이. (abandon은 상태 변경, cleanup은 실제 삭제)
- **Blocker**: 같은 Worktree에서 Resume 할 때 기존 Task Attempt를 재사용할지 새 Attempt를 열지에 대한 정책 확정.

### F. Review, approval, merge
- **Decided**: Owner review 필수, Worker 직접 병합 불가. (Evidence: ADR-0009, ADR-0014)
- **Partially decided**: 사용자 approval이 필요한 시점 (기본은 필요하나 프로필별로 다름). 
- **Undecided**: Approval fingerprint가 포함해야 하는 명확한 필드 목록. (Gap: Fingerprint schema)
- **Recommendation**: Approval fingerprint 설계는 구현 중 추가해도 되는 Non-blocker.

### G. UI / MVP user experience
- **Partially decided**: Desktop-ready Web UI가 첫 목표이며, 설정 화면과 프로젝트 화면 등이 포함됨. (Evidence: Roadmap, ADR-0008, ADR-0010)
- **Undecided**: 실패한 worktree를 UI에서 어떻게 보여주고, Resume/Retry 버튼이 어떻게 노출되는지. (Gap: Error & Recovery UX)
- **Blocker**: Owner-led planning 및 Task 상태를 브라우저 UI에서 어떻게 시각화할지 최소 범위 정의 (ADR-0016 대상).

### H. Acceptance criteria
- **Decided**: Personal Mode Complete 기준, E2E 검증, Opt-in AGY 테스트. (Evidence: ADR-0015, Roadmap)
- **Partially decided**: Golden path 시나리오가 V1을 그대로 복사하지 않는다고 했지만, V2용 새 시나리오가 아직 작성되지 않음.
- **Recommendation**: V2용 Golden Path E2E 시나리오는 테스트 코드 작성 직전에 정의하면 됨 (Non-blocker).

## Conflict Finding
- **Tailscale is required for default Personal Mode app access**: 
  - `ADR-0008`에는 "개인 모드 MVP의 원격 접속은 Tailscale 네트워크를 기본 전제로 한다"고 쓰여 있으나, 
  - `02 Architecture/System Context.md`에는 "Tailscale은 기본 앱 접속 전제가 아니라... 선택적 사설 네트워크 후보다"라고 명시되어 있음.
  - **Gap**: Tailscale이 앱 접근 자체의 기본 요건인지, 아니면 원격 러너/원격 접근 시의 선택지인지 명확히 정리 필요.

## Required Decisions Before Continuing MVP Implementation (Blockers)
1. **Task Decomposition UX**: Owner가 사용자의 넓은 요청을 어떻게 Task로 나누고 UI에 반영할 것인가. (ADR-0016)
2. **Default Write Scope**: Task 생성 시 적용할 기본 write_scope의 입도(Granularity). (ADR-0016/0018)
3. **Resume Semantics**: 실패/타임아웃된 Worktree를 재개(Resume)할 때, 같은 Task Attempt인지 새 Attempt인지의 확정. (ADR-0017)
4. **Tailscale Requirement Conflict**: Tailscale 필수 여부에 대한 정책 충돌 해소.

## Recommended ADR-0016 Scope
ADR-0016 (Personal Mode Complete UX and Owner Conversation Flow)에서는 다음을 다뤄야 한다:
- 최소 UI 화면 목록과 Task 시각화 방식
- Owner의 Task Decomposition (Work Item/Task 분할 로직)
- 기본 Write Scope 및 사용자에게 승인을 요청하는 화면 구조

## Recommended Design Updates Outside ADR-0016
- ADR-0008의 Tailscale 관련 문구를 System Context.md에 맞게 "선택적 보안 접근 채널"로 완화하거나, System Context를 "원격 접속 시 필수"로 통일하는 수정이 필요하다.
- ADR-0017 (Failure Recovery)에서 Resume/Retry 시 Attempt 처리 방식을 확정해야 한다.

## Open Questions to Add or Keep
감사 중 발견된 질문들은 이미 `10 Open Questions/Open Questions.md`의 `## Personal Mode Completion Open Questions` 섹션에 등록되어 있다. (이전 단계에서 추가됨)
- "How should failed worktrees be resumed?" (Attempt 재사용 여부)
- "What is the minimum UI needed for Owner-led planning?"
- "How broad may Owner-assigned write_scope be without explicit approval?"
추가로 등록이 필요한 사항:
- "Does Tailscale serve as a hard requirement for all remote access, or strictly an optional capability?" (추가 예정)

## Non-blockers
- 구체적인 Approval fingerprint 필드
- WorkerRun의 세부 상태값 (DB 스키마)
- Golden Path E2E 테스트의 세부 스크립트 작성
- 데스크톱 앱 포장 기술 (Tauri vs Electron)
이 항목들은 MVP 핵심 로직 구현 과정 중이나 후반부에 확정해도 무방하다.
