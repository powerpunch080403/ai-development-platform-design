# ADR-0025 Worker Identity, Runtime Adapter, and Execution Queue

## Status
Proposed

## Context

Owner can create multiple Tasks and can provide feedback across multiple Tasks. Workers execute Tasks through runtime adapters such as local CLIs or APIs.

The platform must support concurrent User-Owner conversation while Workers run in the background. It must also prevent local CLI/API runtime conflicts, repository conflicts, and ambiguous execution records.

Recent AGY dogfood confirmed that a Worker can execute through Owner ToolCall handoff, produce a ProcessRun, commit a worktree result, and complete a TaskAttempt. This makes Worker identity, adapter binding, capacity, and queue policy necessary design concerns.

## Decision

A Worker is a persistent execution actor bound to exactly one runtime adapter kind.

A Worker is not the CLI/API itself. A Worker is the platform actor that uses a specific CLI/API/runtime adapter under platform authority.

```text
Worker
  - id
  - adapter_kind: agy_cli | codex_cli | openai_api | anthropic_api | mock | manual | future_adapter
  - display_name
  - capabilities
  - max_parallel
  - status
  - runtime_config_ref
```

A WorkerRun is one execution of one Worker against one TaskAttempt.

A ProcessRun or APIRun is the actual external runtime invocation record.

```text
Task
  └─ TaskAttempt
       └─ WorkerRun
            └─ ProcessRun or APIRun
```

Examples:

```text
Worker: Local AGY Worker
adapter_kind: agy_cli
max_parallel: 1

WorkerRun: Local AGY Worker executes README Task Attempt 1
ProcessRun: agy.exe invocation
```

```text
Worker: GPT API Worker
adapter_kind: openai_api
max_parallel: 3

WorkerRun: GPT API Worker executes Summary Task Attempt 1
APIRun: OpenAI API call record
```

## One Worker, One Adapter

Each Worker is bound to one runtime adapter kind.

Good:

```text
Local AGY Worker → agy_cli only
OpenAI API Worker → openai_api only
Anthropic API Worker → anthropic_api only
```

Avoid:

```text
One Worker dynamically switches between AGY, Codex, OpenAI, and Anthropic.
```

Reason:

- audit is clearer
- cost attribution is clearer
- logs are clearer
- queue/capacity is easier
- failure causes are easier to diagnose
- adapter-specific policy is enforceable

Multiple Workers may share the same adapter kind later if the runtime is proven safe for parallelism.

Example:

```text
Local AGY Worker 1
Local AGY Worker 2
```

Personal Mode should start with one local AGY Worker and conservative capacity.

## Owner Concurrency

The Owner must be able to continue User-Owner conversation while Workers are running.

Worker execution must not block the Project Thread.

Recommended flow:

```text
User → Owner in Project Thread
Owner → Worker handoff in Task Work Room
Worker executes asynchronously
Owner returns to Project Thread
Worker events enter Owner inbox/event stream
Owner reacts on the next relevant turn/event
```

Owner should treat both User messages and Worker events as inputs.

```text
Owner input sources:
- Project Thread user messages
- Worker reports
- Worker questions
- WorkerRun completion events
- WorkerRun failure events
- TaskAttempt review events
```

The Owner may provide batch feedback across multiple Tasks. The platform decomposes that feedback into per-Task actions and records each action in the corresponding Task Work Room.

## Worker Execution Queue

Worker execution must pass through a WorkerExecutionQueue or equivalent scheduling boundary.

The queue is responsible for:

- enforcing Worker capacity
- preserving same-Task ordering
- preventing duplicate active attempts
- applying worktree locks
- applying repository apply locks
- recording queued/running/terminal states
- ensuring WorkerRun state is durable

Owner may create multiple Tasks or follow-up attempts quickly. This does not mean all local CLI processes run immediately.

```text
Owner feedback:
Task A → follow-up
Task B → follow-up
Task C → follow-up

AGY Worker capacity = 1

Execution:
Task A running
Task B queued
Task C queued
```

## Default Personal Mode Capacity

Personal Mode defaults should be conservative.

```text
Local AGY Worker max_parallel = 1
Local Codex CLI Worker max_parallel = 1
Manual Worker max_parallel = 1
API Worker max_parallel may be greater than 1 after explicit configuration
```

Reason:

- local CLI tools may share config/cache/session state
- multiple CLI processes may not be safe
- debugging parallel local agents is difficult
- repository/worktree conflicts are easier to avoid with serial local CLI execution

Parallelism can be expanded later after adapter-specific validation.

## Locks and Ordering

Minimum locking policy:

```text
Task lock:
  same Task cannot have multiple active follow-up Attempts.

Task Work Room ordering:
  messages/actions inside the same Task Work Room are ordered.

Worktree lock:
  same worktree cannot have multiple active WorkerRuns.

Repository apply lock:
  applying results to a source repository/default branch is exclusive.

Adapter capacity lock:
  each Worker enforces max_parallel.
```

If file conflict analysis later proves independence across Tasks, execution may be parallelized across different worktrees or repositories. This does not remove per-Worker capacity or per-Task ordering.

## Queue State Guidance

Recommended TaskAttempt states:

```text
created
queued_worker
running_worker
committed
accepted
rejected
failed
cancelled
```

Recommended WorkerRun states:

```text
created
queued
running
succeeded
failed
cancelled
```

`queued_worker` means the Owner has decided a Worker should run, but the execution queue has not started the WorkerRun due to capacity/lock constraints.

`running_worker` means the WorkerRun is actively executing.

Terminal states must close both WorkerRun and TaskAttempt consistently.

## Event Delivery

Worker events are delivered through an inbox/event stream, not by blocking Project Thread execution.

Examples:

```text
WorkerRun.started
WorkerRun.log_available
WorkerRun.question
WorkerRun.succeeded
WorkerRun.failed
TaskAttempt.committed
TaskAttempt.needs_review
```

The Owner consumes these events and may:

- update Task Workspace
- notify User in Project Thread
- request follow-up
- ask User for approval
- accept/reject an Attempt
- choose an apply strategy

## Consequences

Good:

- User-Owner conversation can continue while Workers run.
- Owner can manage multiple Tasks without blocking on one CLI process.
- Local CLI execution stays safe by default.
- Audit and failure attribution remain clear.
- Future API Workers can support higher parallelism without changing core Task model.

Bad:

- Requires queue/scheduler abstraction.
- Requires durable queued state.
- Requires locks and capacity enforcement.
- Some Tasks may wait even when the Owner has already issued feedback.

## Non-goals

- Do not require Temporal/Celery/Dramatiq in MVP.
- Do not define Team Mode remote runner policy here.
- Do not permit Worker to switch adapters dynamically.
- Do not assume local CLI tools are safe for unlimited parallel execution.
- Do not apply results to source/default branch without Owner/User approval.

## Required Invariants

- One Worker is bound to one runtime adapter kind.
- WorkerRun records one Worker execution against one TaskAttempt.
- ProcessRun/APIRun records actual CLI/API runtime invocation.
- User-Owner conversation remains available while Worker runs.
- Worker execution is queued and capacity-controlled.
- Personal Mode local CLI Workers default to `max_parallel = 1`.
- Same Task follow-up attempts are ordered.
- Same worktree has at most one active WorkerRun.
- Applying results to a source repository uses a separate apply lock.
