# ADR-0021 Owner Tool Call Bridge and Authority Envelope Policy

## Status
Accepted

## Context
- ADR-0010 defines Owner Tool Contract and Local Control Plane API.
- ADR-0018 defines Personal Mode approval modes, grants, and autonomy profiles.
- ADR-0019 defines that the application must not replace Owner judgment with keyword routing or hardcoded decision rules.
- ADR-0020 defines Owner Runtime Provider strategy and Codex CLI as the initial MVP Owner provider.
- The next implementation step is Owner Tool Call Bridge.
- Product owner clarified that Personal Mode and Team Mode authority must be separated.

## Decision
Owner Runtime Providers may request platform Tools through Tool Calls.

The platform records Tool Calls, runs execution plumbing, applies the active authority envelope, and records results/audit.

The platform does not replace Owner judgment.

In Personal Mode, Owner decides what files are relevant, what context to inspect, what Tasks to create, and what Tools to request within the user-granted authority envelope.

In Team Mode, the central server/org policy limits what files, context, repositories, tools, secrets, and execution modes are visible or usable by Owner. The primary enforcement mechanism is restricted context/file visibility, not second-guessing Owner's task judgment after the fact.

Owner owns interpretation and planning.
The platform owns authority-envelope application, durable records, and execution plumbing.
Workers own bounded execution only.

## Authority Envelope Principle
The active authority envelope defines what Owner is allowed to see, request, and execute through platform tools.

The platform applies this envelope.
The platform does not judge whether Owner's selected files are product-relevant.
The platform does not reject a Personal Mode Tool Call merely because it independently disagrees with Owner's plan.

## Personal Mode Authority
In Personal Mode, the local user grants authority to Owner.

The user-facing approval modes are:

- Ask for approval
- Approve on my behalf
- Full access
- Custom

Ask for approval:
Owner may plan and request tools, but platform creates approval interruptions when the mode requires user approval.

Approve on my behalf:
Owner may proceed inside the user-granted envelope. Platform records that approval was delegated to Owner.

Full access:
Owner may proceed broadly within the trusted local environment. Platform should not act as a separate file-level product judge. It records actions and applies only explicit user/runtime restrictions.

Custom:
User-defined restrictions form the Personal Mode authority envelope.

If the Personal Mode authority envelope grants access, Owner decides whether to use that access.
The platform should not add an additional product-judgment layer.

## Team Mode Authority
Team Mode will be governed by central server / organization policy.

In Team Mode, central policy may restrict:
- visible projects
- visible repositories
- visible files/directories
- available context
- available tools
- secrets
- network access
- execution modes
- merge/push authority

Team Mode should primarily restrict what Owner can see and use before Owner reasons over the task.

The Team Mode platform should not rely only on rejecting Tool Calls after Owner has already reasoned over forbidden context.

Detailed Team Mode approval and authority profiles are out of scope for this ADR and will be designed later.

## Owner Runtime Provider Responsibility
- receive visible conversation/context/tool definitions
- reason about user intent
- decide what context to inspect
- request Tool Calls
- consume Tool results
- produce assistant messages/status

Provider must NOT:
- directly create Task rows
- directly create WorkerRun rows
- directly mutate Git/worktrees
- directly approve/reject approval requests outside the active authority envelope
- directly merge/push
- directly cleanup worktrees

## Platform Responsibility
The platform:
- records ToolCall requests
- applies the active authority envelope
- executes platform-owned side effects through Tool implementations
- creates approval interruptions when required by Personal Mode profile
- filters/limits context visibility in Team Mode according to central policy
- records ToolCall results, errors, audit events, and policy/authority decisions

## Tool Call Bridge Lifecycle
requested
→ recorded
→ authority_applied
→ approval_required | executing | rejected
→ completed | failed | cancelled

requested:
Owner Runtime Provider requested a Tool.

recorded:
Platform created a durable ToolCall record.

authority_applied:
Platform applied the active authority envelope.

approval_required:
Personal Mode approval profile requires user approval.

executing:
Platform-owned Tool implementation is executing.

rejected:
Request exceeds active authority envelope or invalid tool/schema.

completed / failed / cancelled:
Final durable outcome.

## Tool Call Request Envelope
```json
{
  "agent_run_id": "uuid",
  "provider_kind": "codex_cli",
  "tool_name": "task.create",
  "arguments": {
    "project_id": "...",
    "title": "...",
    "instructions": "...",
    "write_scope": ["..."]
  },
  "provider_call_id": "optional-provider-specific-id",
  "metadata": {
    "source": "owner_runtime_provider"
  }
}
```

- provider-specific fields stay in provider_call_id or metadata
- tool arguments use platform schema
- arbitrary side-effect payloads cannot bypass tool schema

## Tool Call Result Envelope
```json
{
  "tool_call_id": "uuid",
  "status": "completed",
  "result": {
    "task_id": "..."
  },
  "error": null,
  "approval_request_id": null
}
```

Approval required 예시:

```json
{
  "tool_call_id": "uuid",
  "status": "approval_required",
  "result": null,
  "error": null,
  "approval_request_id": "..."
}
```

## Approval Interruption in Personal Mode
When the platform's authority envelope requires explicit user approval in Personal Mode (e.g., under "Ask for approval"), the platform pauses the Tool Call execution and returns a status like `approval_required`.

## Context Visibility in Team Mode
In Team Mode, context restrictions are enforced *before* the Owner reasoning cycle, providing a secure context envelope where only authorized files, histories, and metrics are passed to the Owner.

## Side-effect Boundaries
Owner Runtime Provider can request side effects only through Tool Calls.

The provider cannot directly:
- create Tasks
- create WorkerRuns
- mutate files/worktrees
- merge/push
- cleanup worktrees
- approve its own requests outside the active authority envelope

When Full access or Custom grants broad authority, the platform should not downgrade that authority by adding unrelated file-level product judgment.

## MVP Tool Set
Phase 1:
- read tools
- task.create

Phase 2:
- worker.start_task_attempt
  - worker.start_task_attempt, when implemented later, must create a fresh Worker context for each TaskAttempt.

Phase 3:
- review / approval / merge / cleanup

## Non-goals
- Full definition of Team Mode detailed authority profiles (deferred to future ADR).
- Final finalized schema of all MVP tools (to be defined during implementation).

## Consequences
- The local control plane is clearly scoped to applying the user's authority envelope without overriding logic with separate platform-side task reasoning.
- Clear decoupling of judgment (Owner Runtime Provider) and safe side-effects execution (Platform).
- Safe Personal Mode implementation without unnecessary friction for "Full access" users.

## Related ADRs
- [[07 ADR/ADR-0022 Worker Fresh Context Per Task Policy]]
- ADR-0010 Owner Tool Contract and Local Control Plane API
- ADR-0018 Personal Mode Approval Modes, Grants, and Autonomy Profiles
- ADR-0019 Owner Agent Decision Boundary and Task Infrastructure Policy
- ADR-0020 Owner Runtime Provider Strategy
