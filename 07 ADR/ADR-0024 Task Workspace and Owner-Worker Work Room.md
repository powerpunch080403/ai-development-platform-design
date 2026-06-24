# ADR-0024 Task Workspace and Owner-Worker Work Room

## Status
Proposed

## Context

The platform now has an Owner-led Worker execution path where a Task can produce a Worker result commit through a TaskAttempt, WorkerRun, ProcessRun, worktree, and artifacts.

The next product design question is how users, Owners, and Workers observe and review that work without mixing different conversation scopes.

Important constraints:

- The User talks to the Owner in a project-level conversation.
- The Worker must not take direct instructions from the User.
- The Worker executes Owner-created Tasks and reports results back to the Owner.
- Users need transparency into what happened during a Task.
- Task lists should remain clean even if several follow-up attempts happen under one Task.
- GitHub must not be required for result acceptance or application.

## Decision

A Project has a Project Thread for User-Owner conversation.

A Task has a Task Workspace for task-specific work review.

Inside each Task Workspace, each Task has an Owner-Worker Work Room.

The scopes are separate:

```text
Project
  ├─ Project Thread
  │    └─ User ↔ Owner conversation
  │
  └─ Task Workspace
       └─ Task
            └─ Owner ↔ Worker Work Room
```

The Project Thread is where the User and Owner discuss intent, priorities, approvals, and next steps.

The Owner-Worker Work Room is where the Owner and Worker discuss one specific Task. It contains task-specific instructions, Worker progress, Worker reports, logs, generated files, screenshots, changed files, diffs, result commits, Owner feedback, and follow-up attempt history.

User-Owner conversation is not duplicated into the Task Workspace.

Users may inspect the Owner-Worker Work Room for transparency, but the Worker still does not directly accept User instructions. If the User wants to change a Task, the User tells the Owner in the Project Thread. The Owner decides whether and how to translate that into Worker feedback or a follow-up attempt.

## Task Workspace Contents

A Task Workspace should show task-specific execution results and review material, including:

- Task title and current Task status
- TaskAttempts / work rounds under the Task
- Owner instructions sent to Worker
- Worker reports and questions
- Owner feedback to Worker
- changed files
- diff patch
- git status
- process logs
- stdout/stderr artifacts
- screenshots
- generated files
- result commit SHA
- apply/accept/reject/follow-up history

The review focus is Worker output, not a repeated risk assessment.

Risk and approval are handled before Task creation. The Task Workspace should not make risk level a primary review object. However, violations must still surface clearly, including:

- write scope violation
- grant violation
- policy violation
- unexpected file changes
- failed validation
- execution boundary failure

## Attempts and Follow-up

Follow-up work stays inside the same Task as a new TaskAttempt/work round.

Do not create a new top-level Task for every correction unless the Owner intentionally decomposes the work into a new independent Task.

Recommended UI wording:

- Database / code model: `TaskAttempt`
- User-facing wording: `작업 차수`, `Attempt`, or `Round`

Example:

```text
Task: README 정리
  Attempt 1: rejected
  Attempt 2: committed
  Attempt 3: accepted
```

This keeps the Task list clean while preserving full history.

## Accept, Apply, Reject, Follow-up

Acceptance and application are separate concepts.

- `accept` means the Owner/User accepts the Worker result for this TaskAttempt.
- `apply` means the Owner applies the accepted result through an available strategy.
- `reject` means the current TaskAttempt result is not adopted.
- `follow-up` means the Owner creates another TaskAttempt under the same Task using prior attempt results as explicit context.

Accepting a result must not require GitHub.

Applying a result must not be hard-coded to GitHub PRs. GitHub PR creation is one optional apply strategy, not the default architecture.

Possible apply strategies include:

- create GitHub branch and PR
- cherry-pick result commit into a local branch
- export patch
- apply to a local repository
- defer to manual application
- use another VCS or deployment adapter in the future

The Owner chooses the apply strategy from User intent, repository capabilities, available tools, grants, and policy.

Rejecting an Attempt should preserve diagnostic material:

- logs
- artifacts
- Worker report
- diff
- worktree metadata
- result commit metadata, if present

Rejecting an Attempt does not necessarily reject the whole Task. The Task may remain open for follow-up.

Rejecting the whole Task is a separate Task-level decision.

## State Model Guidance

Recommended Task-level states:

```text
open
in_progress
review
accepted
rejected
completed
cancelled
failed
```

Recommended TaskAttempt-level states:

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

`committed` means the Worker produced a result commit or equivalent result artifact in the execution environment. It does not mean the source repository/default branch was updated.

`accepted` means the Owner/User accepted the Attempt result.

`applied` may be represented either as a separate apply record or as Task/Attempt metadata, depending on implementation stage. It should not be conflated with `committed`.

## Batch Feedback

Owner may perform batch feedback across multiple Tasks from a single Project Thread message.

The platform decomposes batch feedback into per-Task actions and records each action in the corresponding Owner-Worker Work Room.

Example:

```text
User → Owner:
Task A는 채택해.
Task B는 더 짧게 고쳐.
Task C는 폐기해.

Owner actions:
Task A → accept
Task B → create follow-up Attempt with feedback
Task C → reject Attempt
```

Batch feedback must not bypass per-Task ordering, active-attempt limits, adapter capacity limits, or repository/worktree locks.

## Consequences

Good:

- Project-level conversation stays clean.
- Task-specific work history is transparent.
- Worker never receives direct User instructions.
- Follow-up work stays grouped under the original Task.
- GitHub does not become a required product dependency.
- Accept/reject/follow-up flows become explicit and auditable.

Bad:

- UI has more structure: Project Thread, Task Workspace, Work Room.
- Owner must translate User feedback into per-Task Worker actions.
- State model is more precise and therefore slightly more complex.

## Non-goals

- Do not define Team Mode policy here.
- Do not require GitHub for apply/acceptance.
- Do not allow Worker to bypass Owner and talk directly to User.
- Do not require every follow-up to create a new top-level Task.
- Do not define the full implementation schema for all artifacts here.

## Required Invariants

- User-Owner conversation belongs in the Project Thread.
- Owner-Worker work dialogue belongs in the Task Work Room.
- Worker does not directly take instructions from User.
- User may inspect Worker activity for transparency.
- Follow-up attempts remain under the same Task unless Owner intentionally creates a new Task.
- `committed`, `accepted`, and `applied` are distinct concepts.
- GitHub PR is optional apply strategy, not core architecture.
