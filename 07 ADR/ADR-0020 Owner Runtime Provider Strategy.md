# ADR-0020 Owner Runtime Provider Strategy

## Status
Accepted

## Context
- ADR-0006 defines Owner Runtime and AgentRuns.
- ADR-0010 defines Owner Tool Contract and Local Control Plane API.
- ADR-0019 defines that application provides Task/Tool/Policy infrastructure while Owner Agent owns interpretation and planning.
- The initial Personal Mode MVP needs a concrete Owner agent runtime.
- Product owner selected Codex CLI as the initial Owner runtime for MVP.
- The product must remain provider-agnostic and later support API-based and CLI-based Owner runtimes.

## Decision
The product will treat Owner as a domain role and execution responsibility, not as a specific vendor/runtime implementation.

For the initial Personal Mode MVP, the default Owner Runtime Provider will be Codex CLI.

The architecture must support multiple Owner Runtime Providers over time, including:
- CLI-based Owner providers
- API-based Owner providers
- local or remote agent runtimes
- future provider implementations

The platform must not spread Codex CLI-specific assumptions across domain models, Task models, Conversation models, AgentRun records, Tool Call records, approval policy, or UI state.

Owner is the role.
Codex CLI is the first runtime provider.
The platform boundary must remain provider-agnostic.

## Owner Concept vs Runtime Provider
Owner:
- interprets user intent
- decides what context/tools to inspect
- plans Work Items and Tasks
- calls platform tools
- reviews Worker results
- requests approval when needed
- explains outcomes to the user

Owner Runtime Provider:
- executes the Owner agent loop
- receives conversation/context/tool definitions
- returns assistant messages, tool call requests, and run status
- may be implemented through Codex CLI, API calls, or another runtime

## Initial MVP Provider: Codex CLI
Initial MVP will use Codex CLI as the first Owner Runtime Provider.

Codex CLI provider should:
- start from an Owner AgentRun
- receive conversation state and available tool definitions
- operate through the platform Tool Contract
- request tool calls through a controlled bridge
- write AgentRun/Step/ToolCall records
- not bypass approval/grant/policy boundaries
- not directly mutate project state outside platform tools

Forbidden:
- Codex CLI directly writes to application database
- Codex CLI directly starts Workers outside Tool Contract
- Codex CLI directly retries/cleans/merges without platform policy
- Codex CLI-specific fields leak into core domain models unless represented as provider_metadata

## Provider-Agnostic Product Architecture
OwnerRuntimeProvider interface:
- provider_kind
- start_agent_run(agent_run_id)
- resume_agent_run(agent_run_id)
- cancel_agent_run(agent_run_id)
- get_status(agent_run_id)

Provider kinds example:
- codex_cli
- api
- mock
- manual
- future_provider

Provider enum may be needed for initial implementation, but domain logic must not depend on a specific provider.

## Required Provider Boundary
Owner Runtime Provider should only:
- execute Owner reasoning loop
- consume conversation/context/tool schema
- produce assistant messages
- request Tool Calls
- report run status/errors

Provider must not perform:
- direct DB mutation except through platform-owned persistence layer
- direct Worker execution outside Tool Contract
- direct git merge/push
- direct cleanup
- direct approval decision
- direct scope expansion without Owner Tool/Policy records

## API and CLI Provider Support
CLI provider:
- local process execution
- stdin/stdout or file/session bridge
- process timeout/cancel
- provider logs
- provider working directory constraints

API provider:
- request/response or streaming calls
- tool call protocol bridge
- provider run status
- retry/cancel if supported
- provider error mapping

Both API and CLI providers must map into the same platform concepts:
Conversation
AgentRun
AgentRunStep
ToolCall
ApprovalInterruption
PolicyDecision
Task
WorkerRun
Artifact
AuditEvent

## Relationship to Tool Contract
Owner Runtime Provider does not own platform side effects.
The provider may request Tool Calls.
The Local Control Plane validates and executes Tool Calls.
Tool execution must enforce grant, approval mode, write_scope, protected path, danger flag, and audit policy.

## Relationship to Worker Adapters
Owner Runtime Provider:
- runs Owner agent
- decides/plans/tool-calls

Worker Adapter:
- executes bounded Task Attempt
- receives Task/write_scope
- returns result/artifacts/logs
- never decides product intent

Example:
Codex CLI may be used as an Owner Runtime Provider.
AGY may be used as a Worker Adapter.
Future runtimes may be used for either role, but the role boundary must remain explicit.

## MVP Implementation Implications
The next implementation slice after Owner Conversation Input should build an Owner AgentRun Executor Skeleton behind an OwnerRuntimeProvider boundary.

For MVP:
- implement codex_cli provider first
- keep provider metadata isolated
- reuse Conversation and AgentRun records
- reuse ToolCall records
- do not implement keyword-triggered command routing
- do not create Task/WorkerRun directly from user message
- Owner provider must request platform tools instead

## Non-goals
- Full API provider implementation in MVP
- Universal tool protocol standardization across all possible agents
- Removing Codex CLI dependency entirely for MVP

## Consequences
- The application code will remain clean and independent of Codex CLI specifics.
- The UI will render generic AgentRun/ToolCall records regardless of the provider.
- We must build an adapter for Codex CLI that maps its output to the platform Tool Contract.

## Related ADRs
- ADR-0006 Owner Runtime and Agent Runs
- ADR-0010 Owner Tool Contract and Local Control Plane API
- ADR-0019 Owner Agent Decision Boundary and Task Infrastructure Policy
