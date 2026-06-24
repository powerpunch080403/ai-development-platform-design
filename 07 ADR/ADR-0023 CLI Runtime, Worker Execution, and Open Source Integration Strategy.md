# ADR-0023 CLI Runtime, Worker Execution, and Open Source Integration Strategy

## 1. Problem

현재 MVP 구현은 다음을 직접 구현했다.

- Owner ToolCall bridge
- worker.start_task_attempt
- worker.run_task_attempt
- mock completion path
- AGY gate
- BackgroundTasks handoff boundary
- process_runner / antigravity_cli boundary

하지만 장기적으로 다음 문제가 있다.

- FastAPI BackgroundTasks는 durable long-running workflow에 약함
- asyncio subprocess는 interactive CLI/PTY에 약함
- CLI output streaming / cancellation / attach / resume가 약함
- Team Mode / Remote Runner에는 local worktree만으로 부족함
- 외부 agents/tools와 호환하려면 MCP 같은 protocol adapter가 필요할 수 있음
- Policy/authorization은 Team Mode에서 OPA/Casbin 검토 여지가 있음

## 2. Decision

다음 4개 경계를 제품 architecture에 추가한다.

1. WorkerExecutionService
2. ProcessRuntimeProvider
3. ExecutionEnvironmentProvider
4. ToolProtocolAdapter

각 책임:

WorkerExecutionService:
- TaskAttempt / WorkerRun lifecycle orchestration
- start, run, cancel, resume, fail, complete
- BackgroundTasks/Temporal/Dramatiq 같은 executor 선택을 숨김

ProcessRuntimeProvider:
- CLI/process/PTY execution
- stdout/stderr/log streaming
- timeout/cancel
- interactive terminal support in future

ExecutionEnvironmentProvider:
- local worktree
- local container
- remote container
- cloud sandbox

ToolProtocolAdapter:
- internal ToolCall remains source of truth
- future MCP-compatible adapter may expose safe subset of tools

## 3. Open-source evaluation

다음 후보를 평가한다.

Temporal:
- durable workflow engine
- long-running WorkerRun/TaskAttempt에 장기 후보
- 지금 당장은 도입하지 않음

Dramatiq/Celery:
- Python background task queue 후보
- Temporal보다 가벼운 중기 후보
- 지금 당장은 도입하지 않음

node-pty + xterm.js:
- interactive CLI / web terminal 후보
- Desktop/web terminal 단계에서 검토
- 지금 당장은 도입하지 않음

pexpect/ptyprocess:
- Python interactive CLI automation 참고 후보
- Windows/TTY 제약 때문에 제한적
- 지금 당장은 도입하지 않음

MCP:
- external agent/tool protocol adapter 후보
- internal ToolCall/Grant/Audit 모델 대체 아님
- 지금 당장은 도입하지 않음

OpenHands:
- AI coding agent platform/sandbox/session 참고 architecture
- 제품 코어로 통째 도입하지 않음

E2B/Daytona/devcontainer:
- Team Mode / Remote Runner / sandbox 후보
- 지금 당장은 도입하지 않음

OPA/Casbin:
- Team Mode central policy 후보
- Personal Mode policy는 현재 직접 모델 유지

OpenTelemetry:
- runtime observability 보완 후보
- AuditEvent/ToolCall 대체 아님

## 4. Current product-owned domains

아래는 오픈소스로 대체하지 않는다.

- Owner role
- Worker role
- Task / WorkItem
- TaskAttempt
- WorkerRun
- ToolCall durable record
- Grant / Approval / Authority envelope
- AuditEvent
- Worktree preservation policy
- Fresh Worker Context invariant

## 5. Migration strategy

단계별 계획:

Stage 0:
- Current MVP remains operational.

Stage 1:
- Fix Worker.device_id FK bug.
- Introduce WorkerExecutionService interface.
- Move direct BackgroundTasks usage behind LocalBackgroundWorkerExecutionService.
- Keep existing tests passing.

Stage 2:
- Introduce ProcessRuntimeProvider interface.
- Existing process_runner becomes NonInteractiveSubprocessRuntimeProvider.
- antigravity_cli uses runtime provider instead of direct process assumptions where practical.

Stage 3:
- Add log streaming / cancellation / attach APIs.
- Evaluate xterm.js / node-pty for interactive terminal.

Stage 4:
- Evaluate Temporal or Dramatiq for durable WorkerExecutionService backend.

Stage 5:
- Evaluate MCP-compatible ToolProtocolAdapter.
- Keep internal ToolCall as source of truth.

Stage 6:
- Evaluate remote/container ExecutionEnvironmentProvider for Team Mode.

## 6. Immediate implementation rule

앞으로 구현 지시에는 다음 규칙을 적용한다.

- Do not add direct subprocess calls in owner_tools.py.
- Do not add long-running business logic directly to FastAPI endpoints.
- Do not couple core execution logic to BackgroundTasks directly.
- Use service boundary even if implementation is local/simple.
- Tests must verify no real AGY/Codex CLI runs unless explicitly manual.
- Any switch to Temporal/node-pty/MCP/OpenHands requires separate approval.

## 7. Consequences

좋은 점:

- 지금 MVP를 버리지 않고 장기 구조로 확장 가능
- 오픈소스 도입 시 교체 지점이 명확함
- Owner/Worker 제품 철학 유지
- Team Mode / Remote Runner / Terminal UI로 확장 가능

나쁜 점:

- 단기 구현량 증가
- interface가 먼저 생겨 다소 추상화가 늘어남
- 오픈소스 도입 타이밍을 계속 재평가해야 함
