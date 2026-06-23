# ADR-0019 Owner Agent Decision Boundary and Task Infrastructure Policy

## Status
Accepted

## Context
- ADR-0006 defines Owner Runtime as Conversation, Agent Run, Step, Tool Call, Approval Interruption, and durable execution records.
- ADR-0010 defines Tool Contract and Local Control Plane boundaries.
- ADR-0016 defines Owner-created Tasks and Worker write_scope constraints.
- ADR-0017 defines conversation-led recovery.
- ADR-0018 defines Owner grants and approval modes.
- Product owner clarified that the application should not implement the Owner's judgment logic as keyword-triggered rules.
- The application should provide the task operating structure, not replace Owner reasoning with hardcoded command parsing.

## Decision
The application must not implement keyword-triggered command routing as the primary Owner behavior.

User messages are natural-language intent inputs to Owner AgentRuns.
They are not direct application commands.

**Owner Runtime Provider**:
The Owner Agent may be powered by Codex CLI in the initial MVP, but the decision boundary remains the same: the platform provides structure and tools; the Owner agent runtime decides which tools to call.

## Application Responsibilities
The application provides:
- Conversation storage
- AgentRun storage and execution shell
- Tool Contract
- Tool Call records
- Task / Work Item / Task Attempt structure
- write_scope / read_scope / forbidden_scope enforcement
- Project working scope
- Grant / approval mode / policy checks
- WorkerRun / ProcessRun / Worktree / Artifact records
- Review / approval / merge / cleanup bounded operations
- Audit and policy decision records

The platform owns structure and enforcement.

## Owner Agent Responsibilities
The Owner Agent decides:
- what the user likely wants
- what context to inspect
- whether to ask clarification
- whether to create Work Items or Tasks
- what Task scope to assign
- whether to request approval
- which tool to call
- how to interpret Worker result/failure
- whether to retry, continue, abandon, or recommend cleanup

Owner owns judgment. Platform applies the active authority envelope and records Tool/Task execution state.
The Owner Agent owns interpretation and planning.
Workers own bounded execution only.

## Prohibited Keyword-triggered Routing
Do not implement:
- if message contains "retry" or "다시": retry_attempt()
- if message contains "continue" or "이어서": continue_attempt()
- if message contains "cleanup" or "정리": cleanup_worktree()
- if message contains "run" or "실행": start_worker()
- if message contains "fix" or "고쳐": create_task()

Allowed structure:
- store user message
- create Owner AgentRun
- provide available tools and current project/task/runtime context
- let Owner Agent decide whether to call create_task, request_approval, inspect_worktree, start_worker, cleanup_worktree, etc.
- enforce Tool Boundary and Policy before side effects

## Tool and Task Infrastructure
These tools are not decision rules.
They are bounded capabilities that Owner Agent may call.

Application provided Tools:
- project.list
- repository.get_status
- conversation.append_message
- agent_run.create
- task.create
- task_attempt.create
- worker.start_run
- worktree.inspect
- worktree.get_diff
- artifact.list
- approval.request
- approval.record_decision
- cleanup.request
- merge.prepare_squash
- merge.perform_squash

## Conversation Input Semantics
- User message is stored as conversation input.
- User message may start an Owner AgentRun.
- Owner AgentRun receives available context/tools/policy state.
- Application may provide UI affordances, but UI buttons and keywords are not the primary reasoning mechanism.
- A later UI may offer shortcuts, but shortcuts must call the same Tool/Policy/Task infrastructure and must not bypass Owner/Policy boundaries.

## Recovery and Continuation Semantics
When the user says "다시 해줘", "이어서 해줘", "정리해줘", or similar:
- the application must not directly map the phrase to retry/continue/cleanup;
- the message should enter Owner conversation;
- Owner Agent inspects relevant TaskAttempt, WorkerRun, Worktree, diff, logs, artifacts, and policy state if it chooses;
- Owner Agent then decides whether to create a new Attempt, create a follow-up Task, request approval, recommend cleanup, or ask clarification.

A dedicated retry/cleanup button may exist later, but it must be an explicit UI action and still go through Tool Boundary, Policy, Grant, and audit records.

## Consequences
- The backend application remains a generic control plane without hardcoded logic loops for LLM planning.
- Owner Agent becomes completely responsible for the UX orchestration logic.
- UI development will focus on feeding complete state to Owner and visualizing what Owner has done (AgentRun steps, tool calls) rather than wiring direct mutation buttons mapped to intent keywords.

## Non-goals
- Defining the exact prompt format or Context Builder for the Owner AgentRun.
- Defining exactly which Worker models to use.

## Related ADRs
- ADR-0006 Owner Runtime and Agent Runs
- ADR-0010 Owner Tool Contract and Local Control Plane API
- ADR-0016 Initial Personal Mode MVP Execution Policy
- ADR-0017 Failure Recovery and Worktree Resume Policy
- ADR-0018 Personal Mode Approval Modes, Grants, and Autonomy Profiles
