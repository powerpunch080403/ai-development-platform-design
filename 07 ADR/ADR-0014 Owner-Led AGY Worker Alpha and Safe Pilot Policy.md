---
type: adr
status: accepted
date: 2026-06-23
scope:
  - personal-mode
  - mvp
  - policy
  - owner
  - worker
  - pilot
---

# ADR-0014 Owner-Led AGY Worker Alpha and Safe Pilot Policy

## Status
Accepted

## Context
- Personal Mode MVP는 Owner-led 구조다.
- Worker는 독립 Agent가 아니라 Task 실행자다.
- Real AGY Worker Alpha 검증이 진행되었다.
- 구현 검증은 설계에 반영되어야 하며, 이후 구현은 설계를 따른다.
- "설계 먼저, 구현 나중" 원칙을 다시 명확히 한다.

## Decision
제품 방향과 상태 머신, 권한, Owner-Worker 관계는 다음과 같이 결정된다.

- **User ↔ Owner ↔ Worker 관계**: 사용자는 Owner와 대화하며, Worker와 직접 대화하지 않는다. Worker는 사용자와 소통하거나 직접 승인을 요청하지 않는다. Owner가 사용자 요청을 분석하여 Task instruction, write_scope, risk, required capability, approval requirement를 생성한다. Worker는 Task 없이 독립 실행하지 않으며, 임의로 Task를 벗어나는 작업을 하지 않는다.
- **User may give broad natural-language requests to Owner, but Worker must not receive user free-form prompts directly.** 사용자의 "알아서 고쳐줘" 같은 요청은 Owner에게 허용되나, Worker에게는 Task instruction만 전달되어야 한다.
- **write_scope is assigned by Owner per Task and enforced before result commit.** Worker는 write_scope를 임의 확장/변경할 수 없으며, 범위를 벗어난 파일 수정/생성은 result commit 이전에 차단된다.
- **Worker never asks the user for approval; Owner requests approval through Approval Policy.** 실행 전 또는 병합 전 승인은 모두 Owner가 Approval Policy에 따라 진행한다.
- **Preflight is Owner's execution readiness check, not a mandatory user confirmation button for every Worker run.** Owner는 Worker 실행 전 repo 상태, Task 범위, write_scope, grants 등을 점검한다. 위험도가 낮고 Grant로 허용되면 매번 사용자에게 묻지 않는다.
- **Failed, timed out, interrupted, or scope-violating worktrees are preserved by default.** timeout, Worker failure, write_scope violation 시에도 원인 분석이나 이어서 작업을 위해 worktree를 보존하며, 자동 cleanup 하지 않는다.
- **Real AGY Worker Alpha is opt-in and controlled; it is not yet a general free-form AI worker for arbitrary user projects.** 현재 AGY Worker는 제한적인 controlled mode (예: `controlled_readme_test`, `controlled_scope_violation_test`, `controlled_timeout_test`)로만 구동되며 danger flag 기본값은 false이다. Process Runner는 background process의 stdin을 DEVNULL로 닫아 무한 대기를 방지한다.
- **First user-project-like pilot should use a separate pilot repository.** 구현 저장소나 중요 실제 저장소가 아닌 별도 pilot repository (예: `ai-development-platform-agy-pilot`)에서 먼저 사용 흐름을 검증한다.

## Consequences
- Owner planning becomes more important. Owner의 Task 생성, write_scope 및 risk 판단의 정교함이 요구된다.
- Task creation must include write_scope/risk/capability metadata.
- Worker adapters remain simpler and safer. Worker 어댑터는 권한 밖의 범위를 신경쓰지 않고 제한된 Task만 수행한다.
- Failed worktrees may consume disk until user/Owner cleanup.
- Pilot repo is required before important real repo adoption.

## Non-goals
- No free-form Worker prompt.
- No arbitrary command runner.
- No direct Worker-user chat.
- No automatic cleanup of failed worktrees.
- No direct main/default merge by Worker.
- No Team Mode policy changes in this ADR.
- No Remote Runner design changes in this ADR.
- No implementation repository changes in this ADR.

## Related ADRs
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]
- [[07 ADR/ADR-0012 Remote Test Runner Worker Capability]]
- [[07 ADR/ADR-0013 MVP Implementation Slice and Repository Strategy]]
