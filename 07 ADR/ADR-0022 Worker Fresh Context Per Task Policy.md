# ADR-0022 Worker Fresh Context Per Task Policy

## Status
Accepted

## Context
- Worker exclusively executes Tasks created by the Owner.
- Worker does not communicate directly with the User.
- Worker cannot autonomously expand its scope.
- Long-running implicit Worker contexts introduce risks such as hidden memory, scope drift, unauditable decisions, and degraded reproducibility.
- Task execution boundaries must be clearly isolated in both Personal Mode and Team Mode.

## Decision
- Every WorkerRun/TaskAttempt starts with a fresh AI context window.
- Worker runtime providers/adapters must not reuse previous Worker conversation context.
- Previous attempt information may only enter the new context if Owner explicitly includes it in the Task packet or context bundle.
- Worker context is fresh by default and per Task/Attempt.
- Worker memory is not implicit.
- Continuity is explicit and Owner-authored.
- Fresh Worker Context is separate from worktree continuity.
- A resumed Attempt may reuse a preserved worktree, but the Worker AI context still starts fresh.

## Consequences
- Better isolation, auditability, reproducibility.
- Slightly higher context preparation burden for Owner.
- Owner must summarize or attach previous attempt state when continuation is needed.
- Worker cannot rely on hidden memory.

## Non-goals
- Do not implement Team Mode policy here.
- Do not remove failed worktree preservation.
- Do not require every retry to start from a clean filesystem worktree.
- Do not allow Worker to fetch hidden prior context on its own.

## Required Invariants
- WorkerRun has no implicit prior conversation memory.
- Worker cannot read previous Attempt logs unless Owner/platform explicitly provides them through allowed context.
- If a previous worktree is reused, that is a filesystem/workspace decision, not AI memory reuse.
- Any previous state provided to Worker must be recorded as part of Task/Attempt context.
