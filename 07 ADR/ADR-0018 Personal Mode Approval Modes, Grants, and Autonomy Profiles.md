---
type: adr
status: accepted
date: 2026-06-23
scope:
  - personal-mode
  - mvp
  - approval
  - grants
  - autonomy
---

# ADR-0018 Personal Mode Approval Modes, Grants, and Autonomy Profiles

## Status
Accepted

## Context
- ADR-0016 introduced project working scope, Task write_scope, and approval mode hooks.
- ADR-0017 clarified that failure recovery is Owner conversation-led.
- Product owner feedback clarified that Owner autonomy should be stronger than the initial conservative model.
- The user gives approval/grants to Owner, not directly to Worker.
- Worker remains constrained by Owner-created Tasks and cannot expand its own scope.
- Initial MVP needs a practical approval model similar in spirit to Codex-style approval settings, but with explicit Task structure.

## Decision
We define the 4 approval modes, the concept of Owner grants, the relationship between project working scope and Task write_scope, scope expansion rules, protected path handling, Owner explanation responsibilities, and the initial MVP baseline for Personal Mode.

## Approval Modes
We define the following 4 approval modes:

### 1. Ask for approval
- Owner asks the user before actions that cross configured boundaries.
- Good default for cautious users.
- Required for risky/broad actions unless explicitly changed.
- Worker still never asks the user directly.

### 2. Approve on my behalf
- User delegates approval authority to Owner within a configured project working scope and risk envelope.
- Owner may approve routine scope expansions or repeated prompts on behalf of the user.
- Owner must still explain important or risky actions in conversation or audit records.
- Protected/high-risk areas may still require explicit user approval depending on policy.

### 3. Full access
- User gives Owner broad authority within the selected project/repository context.
- Owner may plan Tasks, approve scope expansions, and continue work with minimal interruption.
- This does not mean Worker has unrestricted independent authority.
- Worker still acts only through Owner-created Tasks.
- Owner must still prevent obviously dangerous, secret-leaking, or source-repo-destroying behavior.

### 4. Custom
- User defines custom rules by project, path, risk category, action type, or adapter.
- Custom can be stricter or looser than the default profiles.
- Detailed custom rule UI may be incremental, but the concept is part of the model.

## Grants
A grant is a user-approved authorization that allows Owner to perform or approve certain actions under defined boundaries.

A grant may be connected to:
- project
- repository
- project working scope
- path category
- action type
- risk level
- time/session
- adapter/capability

Core principles:
- User grants authority to Owner.
- Owner converts that authority into Task constraints for Worker.
- Worker never receives open-ended authority directly from User.
- Worker cannot create, expand, or reinterpret grants.
- Owner must be able to explain which grant allowed an action.

Initial MVP minimum grant fields (conceptual):
- `approval_mode`
- `project_id`
- `repository_id`
- `allowed_project_working_scope`
- `allowed_risk_ceiling`
- `allow_scope_expansion`
- `allow_new_files`
- `allow_protected_paths`
- `allow_high_risk_changes`
- `allow_danger_flag`
- `expires_at` or session_bound

Note: This ADR does not finalize the exact DB schema for grants.

## Project Working Scope and Task Write Scope
Project working scope governs Owner planning boundaries, whereas Task write_scope governs Worker mutation boundaries.

### Project working scope
- Optional project-level boundary for Owner activity.
- Can be configured during project creation or later.
- Can include existing files/folders.
- Can include planned files/folders.
- Can be left unset.
- Unset means no project-level scope restriction has been configured yet.
- Unset does not let Worker act without Task write_scope.

### Task write_scope
- Concrete per-Task mutation boundary assigned by Owner.
- Passed to Worker as part of the Task.
- Worker must stay within Task write_scope.
- Task write_scope should normally fit inside project working scope when one exists.
- Owner may request or apply approval for scope expansion when needed.

## Scope Expansion
If desired work requires files/actions outside the current project working scope or Task write_scope:
- Owner decides whether this is routine, risky, or blocked.
- Depending on approval mode/grant, Owner either:
  1. asks the user,
  2. approves on the user's behalf,
  3. proceeds under Full access,
  4. follows Custom rules.

Owner should explain:
- why scope expansion is needed,
- which files/categories are involved,
- what risk is introduced,
- whether the active approval mode allows automatic approval.

Worker limits:
- Worker cannot expand scope.
- If Worker modifies outside Task write_scope, result is `scope_violation`.
- Owner may create a new Task with expanded write_scope if approval/grant allows.

## Protected Paths and High-risk Changes
Protected paths are not strictly forbidden, but require Owner intent and policy/approval.

Protected examples:
- `.env`, `.env.*`
- secrets, credentials, private keys
- local DB files
- runtime-data
- worktrees
- artifacts
- sensitive logs

Policy:
- Worker must not touch protected paths unless Owner explicitly includes them in Task scope and policy/grant allows it.
- Protected path access should be rare and explained.
- Full access may reduce repeated prompts, but does not silently allow secret exfiltration or destructive behavior.
- If protected access risks exposing secrets to external tools/models, Owner must block or ask explicit approval depending on model/tool constraints.

High-risk examples:
- authentication/authorization code
- security policy code
- dependency manifests or lock files
- database migrations
- CI/CD configuration
- deployment configuration
- shell/executable scripts
- package manager configuration
- broad runtime behavior changes

Policy:
- High-risk changes require Owner risk classification.
- "Ask for approval" mode should ask the user.
- "Approve on my behalf" may allow routine high-risk categories only if configured.
- "Full access" allows more autonomy but still requires Owner explanation/audit.
- "Custom" rules override according to user configuration.

## Adapter / Capability
Adapter is the connector that translates a platform Task into a concrete execution method and translates the result back into platform records.
(어댑터는 플랫폼의 Task를 실제 실행 방식으로 연결하는 번역기다. 예: Manual Worker, Mock Worker, AGY CLI, future Codex CLI.)

Policy:
- Owner chooses adapter/capability when creating or routing a Task.
- User normally does not need to manage adapter details.
- Some adapters may require stricter approval due to external model/tool behavior.
- Danger flags and external CLI permissions are governed by approval mode, grants, and local config.

## Owner Explanation Responsibilities
Owner must explain or record the following:
- scope expansion
- protected path access
- high-risk change
- danger flag use
- external CLI/tool capability use
- merge to default/source branch
- cleanup of preserved failed worktree
- proceeding under Full access for a broad or risky action

Format:
- conversation explanation,
- approval request,
- audit event,
- policy decision record,
- or a combination.
For initial MVP, conversation explanation and audit/policy logs are sufficient.

## Initial MVP Policy
- Default approval mode should be **Ask for approval** unless the user selects otherwise.
- Project working scope may be unset.
- Task write_scope remains required for Worker execution.
- Owner may infer Task write_scope from user request, project context, and project working scope.
- Owner may ask for approval or auto-approve based on active approval mode.
- Approve on my behalf and Full access may exist as settings even if early UI is simple.
- Custom may initially be minimal but must be represented in the design.
- AGY or real AI Worker use is an important MVP checkpoint.
- External AI Worker paths may require more conservative defaults than Manual/Mock.
- Danger flag use must never be enabled only by arbitrary request body input.
- Danger flag requires local config and grant/policy allowance.
- Merge to default/source branch remains Owner-mediated. Depending on approval mode, Owner may ask the user or proceed under grant. Worker never merges.

## Consequences
- The clear definition of approval modes aligns the system with user expectations of a Codex-style app while preserving the underlying Task safety net.
- Owner autonomy is decoupled from Worker autonomy, allowing Owner to handle routine expansion while preventing rogue Worker changes.
- Requires robust Owner capability to classify risk and explain boundaries.

## Non-goals
- Exact DB schema and migration for Grants table.
- Detailed UI/UX design for custom rule settings.
- Team Mode approval inheritance rules.
- Remote Runner detail changes.

## Related ADRs
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0014 Owner-Led AGY Worker Alpha and Safe Pilot Policy]]
- [[07 ADR/ADR-0016 Initial Personal Mode MVP Execution Policy]]
- [[07 ADR/ADR-0017 Failure Recovery and Worktree Resume Policy]]
