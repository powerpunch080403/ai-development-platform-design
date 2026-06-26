# Personal Mode Architecture Hardening Roadmap

## Status

Proposed implementation roadmap.

This document is the persistent reference for the next implementation sequence in `powerpunch080403/ai-development-platform`.

It exists because implementation conversations can exceed a single assistant context window. Future sessions should use this document as a compact, durable handoff anchor before changing code.

## Current baseline

As of implementation `main` after PR #27 through PR #32:

- Personal Mode web shell and three-pane workspace are merged.
- Radix UI primitives and Antigravity-style inspector review UI are merged.
- Provider-aware `AgentRun` metadata and central Owner run status visibility are merged.
- The implementation repo has Local Runtime, Conversation, AgentRun, ToolCall, Task, TaskAttempt, WorkerRun, Worktree, Review, Approval, and squash-merge baseline records.
- Codex CLI Owner Runtime Provider is still a skeleton/prompt bridge. Real Owner reasoning/tool-loop is pending.
- AGY Worker path exists as a controlled, opt-in worker alpha. Free-form worker prompt exposure remains out of scope.

The next product milestone is not more UI polish. The next milestone is a durable, provider-agnostic Owner ToolCall loop that creates Tasks through the platform Tool Contract.

## Product goal

The long-term goal is a complete Personal Mode where the user can say what they want, and the system can:

1. understand the project context,
2. plan work as Owner,
3. create Tasks through durable ToolCalls,
4. start fresh Worker attempts,
5. inspect diffs and artifacts,
6. explain outcomes,
7. request or apply approval according to the user's autonomy profile,
8. squash merge safely,
9. recover from failure,
10. remember project/user decisions with clear scope and provenance.

## Non-negotiable architectural rules

1. Owner is a domain role, not a vendor runtime.
2. Owner Runtime Provider executes the Owner reasoning loop.
3. Provider-specific details stay in provider metadata, adapter code, or provider-specific error mapping.
4. Platform state remains durable in the app database: Conversation, AgentRun, AgentRunStep, ToolCall, Task, TaskAttempt, WorkerRun, Artifact, Review, Approval, AuditEvent.
5. Owner Runtime Provider must not directly create Tasks, WorkerRuns, worktrees, approval records, merge commits, or cleanup side effects.
6. Provider side effects must go through ToolCall records and the active authority envelope.
7. UI renders generic records, not Codex-specific or AGY-specific branches.
8. Keyword routing from user message to platform side effects is prohibited. Owner reasoning may request tools; the platform must not infer Tasks from keywords.
9. Prefer long-term architectural correctness over short-term convenience.
10. Any change to data model, provider boundary, authority behavior, workflow ordering, or product direction needs an explicit user decision before implementation.

## Context-window operating protocol

Each new implementation session should start by reading, in this order:

1. this roadmap,
2. implementation repo `README.md`,
3. implementation repo `docs/WORKING_RULES.md`,
4. ADR-0020 Owner Runtime Provider Strategy,
5. ADR-0021 Owner Tool Call Bridge and Authority Envelope Policy,
6. the current implementation branch diff or PR body,
7. the latest local validation output.

Each implementation PR must include a handoff block in its PR body:

```text
Context handoff
- Current branch:
- Base commit:
- Scope:
- Explicitly out of scope:
- Design references:
- Validation run:
- Next recommended slice:
- Known risks:
```

Each slice should be small enough that a new assistant session can recover from only:

- this roadmap,
- the PR body,
- `git log --oneline --decorate -10`,
- `git status --short`,
- targeted test output.

Do not rely on conversational memory for product-critical decisions. Put decisions in design docs, PR bodies, or implementation docs.

## Target server structure

Move gradually toward this structure. Do not perform a big-bang rewrite.

```text
apps/server/src/aidp_server/
  api/
    conversations.py
    tasks.py
    tool_calls.py
    reviews.py
    approvals.py

  application/
    owner_run_service.py
    owner_tool_loop.py
    task_creation_service.py
    task_attempt_service.py
    worker_queue_service.py
    review_service.py
    approval_service.py

  domain/
    authority.py
    tool_contracts.py
    task_policy.py
    write_scope_policy.py
    state_transitions.py
    autonomy_profile.py

  owner/
    context_builder.py
    prompt_builder.py
    tool_result_interpreter.py
    providers/
      base.py
      fake_scripted.py
      codex_cli.py
      openai_api.py
      local_ai.py

  workers/
    queue.py
    liveness.py
    execution_service.py
    adapters/
      mock.py
      manual.py
      antigravity_cli.py

  infrastructure/
    git/
    process_runner/
    mcp/
```

The existing implementation can keep its current file layout while new seams are extracted. Prefer extraction at the point where a feature needs the seam.

## Open-source integration stance

Do not add a full agent framework dependency to the core implementation yet.

Use this rule:

- Core state and lifecycle are owned by this platform.
- Open-source agent frameworks may be used later as provider adapters.
- MCP-compatible concepts may shape the internal ToolCall envelope, but MCP should not own platform state.
- Codex CLI, OpenAI Agents SDK, Pydantic AI, LangGraph, or future runtimes should all map into the same provider boundary.

Near-term implementation should use an internal, minimal, MCP-compatible ToolCall envelope:

```text
OwnerToolRequest
- provider_kind
- agent_run_id
- tool_name
- arguments_json
- provider_call_id
- metadata
```

The platform converts this into durable `ToolCall`, applies authority, executes the platform-owned tool, records result/audit, and returns a generic result envelope to the provider.

## Phase 0 — Design handoff PR

Goal: create this roadmap and use it as the stable reference for implementation.

Acceptance:

- Roadmap exists in design repo.
- Future implementation PRs reference it.
- Context-window protocol is documented.

## Phase 1 — Architecture hardening for Owner ToolCall loop

Recommended implementation PR title:

```text
[codex] Add Owner ToolCall loop foundation for task.create
```

Scope:

1. Add shared Task creation service.
2. Make existing HTTP Task creation use the shared service.
3. Make `owner_tools.task.create` use the same shared service.
4. Add `owner_tool_loop.py` as the provider-facing loop seam.
5. Add fake/scripted Owner provider path that can explicitly request `task.create` through the loop.
6. Record AgentRunStep for ToolCall execution.
7. Preserve no-keyword-routing behavior.
8. Add tests proving ToolCall, Task, AgentRunStep, AuditEvent, and no WorkerRun/Approval side effects.

Out of scope:

- real Codex CLI structured tool loop,
- worker.start_task_attempt from Owner loop,
- AGY free-form worker execution,
- review/approval/merge automation,
- memory,
- UI redesign.

Expected implementation files:

```text
apps/server/src/aidp_server/task_creation.py
apps/server/src/aidp_server/owner_tool_loop.py
apps/server/src/aidp_server/work.py
apps/server/src/aidp_server/owner_tools.py
apps/server/src/aidp_server/owner_providers.py
apps/server/tests/test_owner_tool_loop.py
apps/server/tests/test_owner_providers.py
```

Acceptance tests:

```text
fake/scripted Owner provider starts AgentRun
-> provider requests task.create through owner_tool_loop
-> request_owner_tool_call creates durable ToolCall
-> ToolCall caller_type == OWNER
-> ToolCall caller_id == fake
-> ToolCall status == succeeded
-> ToolCall result_json.task_id exists
-> Task status == DRAFT
-> Task.agent_run_id == AgentRun.id
-> Task.project_id == AgentRun.project_id
-> AgentRun status == completed
-> AgentRun provider_metadata_json.tool_loop_executed == true
-> no WorkerRun created
-> no ApprovalRequest created
```

Negative tests:

```text
unknown tool -> rejected ToolCall
invalid write_scope -> failed ToolCall and no Task
repository from another project -> failed ToolCall and no Task
keyword-only user message -> no Task, WorkerRun, or ApprovalRequest
```

## Phase 2 — Owner context builder and read tools

Goal: Owner receives coherent context instead of ad hoc prompt text.

Scope:

1. Add `owner/context_builder.py`.
2. Build visible project/repository/conversation/task/attempt context.
3. Include available tool definitions.
4. Include authority/autonomy profile summary.
5. Keep context provider-agnostic.
6. Add read-only `project.list`, `repository.list`, `task.list`, and conversation summary paths as needed.

Acceptance:

- Provider receives structured context object.
- Context contains no forbidden provider-specific assumptions.
- Read tools produce durable ToolCall records when requested by Owner.
- Context can be rendered/debugged in tests without running Codex CLI.

## Phase 3 — Codex CLI structured bridge

Goal: connect Codex CLI to the ToolCall loop without letting it mutate platform state directly.

Scope:

1. Define structured stdout protocol for assistant messages and tool requests.
2. Parse provider output into generic events.
3. Support one or more tool-call turns.
4. Feed tool results back to provider if supported by the selected bridge mode.
5. Map provider failures to provider-aware AgentRun errors.
6. Keep prompt mode and bridge spike safe fallbacks.

Out of scope:

- direct DB access from Codex CLI,
- direct git/worktree mutation from Codex CLI,
- free-form worker prompt exposure.

Acceptance:

- Codex CLI provider can request `task.create` through the same `owner_tool_loop` path.
- Quota, timeout, command-not-found, empty-response, and malformed-tool-output errors are recorded durably.
- Provider-specific output stays in provider metadata or adapter logs.

## Phase 4 — Owner starts Worker attempts through tools

Goal: complete `Task -> Attempt -> Worktree -> WorkerRun` from Owner decisions.

Scope:

1. Owner requests `worker.start_task_attempt` through ToolCall.
2. Worker context is fresh per TaskAttempt.
3. Worktree is created through platform service.
4. WorkerRun is queued or started according to worker capacity and adapter availability.
5. Existing liveness and stale recovery remain authoritative.

Acceptance:

- Owner-created Task can become a TaskAttempt and WorkerRun without direct provider side effects.
- WorkerRun records and TaskAttempt transitions are valid.
- No reused stale worker context unless explicitly modeled and allowed by policy.

## Phase 5 — Conversational review, follow-up, approval, and merge

Goal: make the review loop feel like a product, not a raw pipeline.

Scope:

1. Owner summarizes Worker result and diff.
2. User can request follow-up in chat.
3. Owner creates follow-up attempt through ToolCall.
4. Owner requests approval only through approval ToolCall/policy path.
5. User approval validates fingerprint and performs squash merge.
6. Cleanup is explicit and auditable.

Acceptance:

- User can go from request to merged result through the chat-centered UI.
- Follow-up does not create conflicting active attempts.
- Approval fingerprint prevents stale or changed arguments from being approved silently.
- Merge and cleanup are auditable.

## Phase 6 — Autonomy profile and authority envelope completion

Goal: Personal Mode should be powerful without being reckless.

Scope:

1. Implement user-facing autonomy modes:
   - ask_for_approval,
   - approve_on_my_behalf,
   - full_access,
   - custom.
2. Apply authority envelope consistently to Owner tools.
3. Record delegated approval decisions.
4. Make risk-level and approval behavior visible in UI.
5. Add custom restrictions for repositories, paths, tools, execution modes, and merge authority.

Acceptance:

- Side-effect tools either execute, reject, or pause for approval according to the active profile.
- The platform does not second-guess Owner product judgment inside the granted envelope.
- Team Mode policy remains separate and is not mixed into Personal Mode authority.

## Phase 7 — Personal memory and project knowledge

Goal: Owner remembers the user's project preferences and prior decisions safely.

Scope:

1. Add explicit memory records with scope:
   - user,
   - project,
   - repository,
   - conversation,
   - task.
2. Add provenance:
   - source message,
   - source ToolCall,
   - source approval,
   - created_by,
   - confidence,
   - expiry/review policy.
3. Add memory read/write tools.
4. Require user-visible audit for memory writes.
5. Keep memory out of Team Mode until central policy is designed.

Acceptance:

- Owner can reuse stable preferences without relying on hidden conversation context.
- User can inspect, edit, and delete remembered facts.
- Memory never bypasses current project/repository authority envelope.

## Phase 8 — Desktop-ready runtime and multi-session continuity

Goal: make Personal Mode reliable across normal local use.

Scope:

1. Local runtime startup/health UX.
2. Desktop shell packaging path.
3. Runtime session recovery.
4. Background worker/process monitoring.
5. Multi-window/multi-session behavior.
6. Safe shutdown/resume.

Acceptance:

- User can restart the app and continue from durable AgentRun/Task/Attempt state.
- Running/queued WorkerRuns recover safely.
- Worktrees and local process state are not orphaned silently.

## Phase 9 — Reliability, observability, and release readiness

Goal: make the system debuggable enough to trust.

Scope:

1. Structured audit timeline per AgentRun/TaskAttempt.
2. Operations dashboard for active/stale/failed runs.
3. Exportable debug bundle with no secrets.
4. Golden path E2E smoke test.
5. Failure-mode E2E tests:
   - provider quota,
   - provider timeout,
   - worker timeout,
   - write-scope violation,
   - stale approval,
   - merge conflict,
   - cleanup failure.

Acceptance:

- A failed run can be understood without reading raw logs first.
- Recovery action is clear to the user.
- Test suite covers the complete Personal Mode golden path and critical failure modes.

## Per-slice implementation checklist

Every implementation slice should follow this checklist:

```text
1. Read this roadmap and current WORKING_RULES.
2. State current implementation state.
3. State the problem.
4. List viable options if boundary/data/workflow changes are involved.
5. Recommend one option.
6. Ask for explicit user decision when required.
7. Implement the smallest durable slice.
8. Run targeted server tests.
9. Run web build if contracts or UI are touched.
10. Update PR body with context handoff.
11. Keep the PR draft until validation is clean.
12. Merge with squash when approved.
```

## Near-term PR queue

```text
Design PR D1:
- Add this roadmap.

Implementation PR #33:
- Architecture hardening for Owner ToolCall loop and task.create.

Implementation PR #34:
- Owner context builder and read tools.

Implementation PR #35:
- Codex CLI structured tool bridge for task.create.

Implementation PR #36:
- Owner starts Worker attempt through ToolCall.

Implementation PR #37:
- Conversational review/follow-up/approval loop.
```

Do not advance to the next phase when the current phase has weak tests or unclear ownership boundaries.
