# Personal Mode Complete Experience

관련 결정:
- [[07 ADR/ADR-0015 Personal Mode Completion Scope and Design Roadmap]]
- [[07 ADR/ADR-0016 Initial Personal Mode MVP Execution Policy]]
- [[07 ADR/ADR-0017 Failure Recovery and Worktree Resume Policy]]

## Goal
사용자가 로컬 환경에서 실제 프로젝트를 등록하고, Owner와 대화하여 Task를 만들며, Worker가 격리된 worktree에서 안전하게 작업을 수행하고 검토/병합/복구까지 이어지는 완결된 AI 개발 워크스페이스를 제공한다.
개인 모드는 팀 협업이나 중앙 서버 없이 단일 로컬 사용자 한 명을 위해 작동하는 완성된 개발 작업공간이다.

## What the user can do
- 로컬 컴퓨터(Windows/Linux)에서 실행되는 앱을 통해 로컬 프로젝트를 등록하고 관리할 수 있다.
- 브라우저를 통해 앱에 접속하여 Owner AI와 대화하고 작업 계획을 논의할 수 있다.
- 복잡한 작업을 여러 Task로 나누어 Worker에게 위임할 수 있다.
- Worker가 변경한 코드(diff)와 실행한 테스트 결과를 검토하고, 문제없을 경우 기본 브랜치(main/default)로 squash merge를 승인할 수 있다.
- Worker의 작업이 실패하거나 타임아웃, write_scope를 위반하여 중단되었을 때, 그 상태(worktree)를 보존하여 디버깅하거나 작업을 이어나갈 수 있다.

## Owner-led workflow
- **사용자는 Owner와 대화한다.**: 사용자의 모든 자연어 요청(포괄적인 큰 요청 포함)은 Owner에게 전달된다.
- **Owner가 작업을 쪼개고 통제한다.**: Owner는 사용자 요청을 분석해 필요한 경우 승인을 요청하며, 승인 모드와 Project Working Scope에 기반하여 자율적으로 판단하여 구체적인 Task들로 분해한다.
- **Owner explains plan**: Owner는 작업 계획을 사용자에게 설명하고, 필요한 경우 질문(clarification)을 하거나, 승인(approval)을 요청한다.

## Worker execution workflow
- **Worker는 Task만 수행한다.**: Worker는 Owner가 생성한 Task를 할당받아, 지정된 write_scope와 격리된 Git worktree 내에서만 코드를 수정한다.
- **Worker는 사용자와 직접 대화하지 않는다.**: Worker는 채팅 인터페이스를 가지지 않으며, 자유로운 형태(free-form)의 프롬프트를 사용자로부터 직접 받지 않는다.
- 작업 브랜치에 변경 사항을 자동으로 commit하고, diff와 테스트 로그 등 산출물(artifacts)을 생성한다.

## Review and approval workflow
- Owner는 Worker가 남긴 diff와 산출물을 검토하여 승인 정책에 따라 사용자에게 리뷰를 요청한다.
- 사용자는 UI를 통해 변경된 코드, 테스트 결과, 위험 등급 등을 확인한다.
- 사용자가 승인하면, Owner(Local Control Plane)를 통해 기본 브랜치로 squash merge가 진행된다. Worker가 직접 병합하지 않는다.

## Failure and resume workflow
- **실패한 작업은 사라지지 않고 남는다.**: 타임아웃, 프로세스 에러, write_scope 위반 등으로 작업이 실패하더라도 작업 중이던 Git worktree는 기본적으로 보존된다.
- **복구는 대화 중심(Conversational Recovery)이다.**: 사용자는 Owner에게 "다시 해줘", "이어서 해줘", "정리해줘", "왜 실패했어?"라고 요청하며, Owner가 현재 파일 상태와 로그를 분석해 그에 맞는 Task Attempt를 생성하거나 폐기를 권장한다.

## Settings and safety controls
- 로컬 환경 설정, 자율성 프로필(Autonomy Profile), 승인 선호도 등을 조정할 수 있다.
- Danger flag와 프로세스 런너의 `stdin DEVNULL` 규칙, 환경 변수 허용 목록(allowlist) 등 보안 및 안전 장치가 기본 적용된다.

## What is not included
- 다중 사용자 협업 및 팀 모드 기능 (Team Workspace, Approval Group 등)
- 중앙 권한 서버(Central Authority) 및 중앙 DB(PostgreSQL) 연동
- 사용자와 Worker 간의 직접적인 자연어 채팅(free-form UI)
- 원격 자동 배포나 복잡한 클라우드 통합(MVP 범위 외)

## Completion checklist
- [ ] Local Runtime and App Shell 안정화
- [ ] Identity, Device, Session 관리 로직
- [ ] Project and Repository Management
- [ ] Conversation and Owner Experience
- [ ] Work Item / Task / Task Attempt 라이프사이클
- [ ] Owner Tool Contract and Policy Engine
- [ ] Worker and Adapter System (AGY Alpha 통합)
- [ ] Git Worktree and Branch Lifecycle
- [ ] Review, Approval, Merge 흐름
- [ ] Failure, Recovery, Resume, Cleanup 기능
- [ ] Artifact and Audit Trail 시스템
- [ ] Process Runner and Local Command Safety
- [ ] Testing and Validation (E2E)
- [ ] Settings and User Control UI
- [ ] Documentation and Operability (가이드 작성)
