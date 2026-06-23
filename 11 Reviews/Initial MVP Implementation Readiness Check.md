# Initial MVP Implementation Readiness Check

## Purpose
ADR-0014, ADR-0015, ADR-0016, ADR-0017이 제정됨에 따라, 초기 MVP 구현으로 돌아가기 전에 남은 blocker가 있는지 최종 점검한다. 설계가 구현보다 우선하며, 확정된 정책이 구현에서 누락되지 않았는지 판별하는 체크리스트로 동작한다.

## Reviewed Documents
- `11 Reviews/Initial Personal Mode MVP Design Gap Audit.md`
- `07 ADR/ADR-0014 Owner-Led AGY Worker Alpha and Safe Pilot Policy.md`
- `07 ADR/ADR-0015 Personal Mode Completion Scope and Design Roadmap.md`
- `07 ADR/ADR-0016 Initial Personal Mode MVP Execution Policy.md`
- `07 ADR/ADR-0017 Failure Recovery and Worktree Resume Policy.md`
- `07 ADR/ADR-0018 Personal Mode Approval Modes, Grants, and Autonomy Profiles.md`
- `09 Roadmap/Personal Mode MVP Roadmap.md`
- `10 Open Questions/Open Questions.md`
- `01 Product/Personal Mode MVP.md`
- `01 Product/Personal Mode Complete Experience.md`

## Summary Verdict
**Initial Personal Mode MVP implementation can resume.**

Implementation must follow ADR-0014, ADR-0016, ADR-0017, and ADR-0018.
Any new product-direction decision must go back to the design repository first.

*Note: Product owner review after readiness check clarified that Owner autonomy should be stronger than the initial conservative ADR-0016 wording. ADR-0016/0017 were updated to introduce project working scope, approval modes, and conversation-led recovery.*

*Product owner final cutline: Implementation can resume with the target that initial MVP includes Owner/Worker operation, C-level UI state surfaces, and a controlled real AI Worker happy path. Personal Mode Full access/Custom may permit danger-level operations only under explicit local user grant and local configuration. Team Mode authority remains bounded by central policy.*

## Readiness Matrix

### A. Owner/Worker model
- User talks to Owner? - **Ready**
- Worker never talks directly to User? - **Ready**
- Worker executes only Owner-created Task? - **Ready**
- Worker cannot create/expand own Task/write_scope? - **Ready**

### B. Task execution policy
- Worker-executable Task requires explicit instruction? - **Ready**
- explicit write_scope required? - **Ready**
- no implicit whole-repo scope? - **Ready**
- allow_new_files default false? - **Ready**
- protected paths identified? - **Ready**

### C. Preflight
- Owner preflight required before Worker execution? - **Ready**
- preflight is not mandatory user button? - **Ready**
- active WorkerRun guard included? - **Ready**
- preflight recording path sufficient for MVP? - **Ready**

### D. Failure/retry/resume
- failed worktree preserved? - **Ready**
- retry creates new Attempt? - **Ready**
- same Attempt not overwritten? - **Ready**
- manual continuation policy clear? - **Ready**
- abandon vs cleanup clear? - **Ready**

### E. Review/approval/merge
- Owner review required? - **Ready**
- Worker cannot merge? - **Ready**
- squash merge default? - **Ready**
- user approval before default branch merge? - **Ready**
- stale/dirty/approval blocked cases covered? - **Ready**

### F. MVP scope
- Mock/Manual Worker enough for core MVP? - **Ready**
- AGY Alpha opt-in, not required for MVP core? - **Ready**
- External CLI free-form prompt not open? - **Ready**
- Tailscale no longer hard requirement for app access? - **Ready**

### G. UI / acceptance
- minimum MVP UI identified? - **Ready**
- Golden Path still valid? - **Ready**
- remaining open questions non-blocking? - **Ready**

## Remaining Non-blocking Questions
세부 보존/삭제 주기, UI 버튼 vs 명령어의 세부 형태, 긴 실행의 취소 매커니즘 상세, 데스크톱 앱 포장 기술, 아티팩트 압축/삭제 정책 등은 현재 시점에서는 구현을 차단하지 않는 Non-blocking Question으로 유지된다. 구현 과정 또는 후속 ADR에서 다룬다.

## Implementation Can Resume
Initial Personal Mode MVP implementation can resume.

## Recommended Next Implementation Slice
**Owner-Led MVP Execution Alignment**

이 slice의 목적:
- Align current implementation with ADR-0016 and ADR-0017
- Verify explicit write_scope requirements
- Verify allow_new_files default false
- Verify protected path behavior
- Verify failed attempt retry creates new Attempt
- Verify failed worktree preservation
- Verify Tailscale is not assumed for app access
