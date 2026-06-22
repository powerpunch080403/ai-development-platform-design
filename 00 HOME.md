# 설계 홈

## 제품 한 문장

하나의 앱에서 개인 모드와 팀 모드를 제공하고, 개인 실행 환경의 Owner AI와 Worker가 작업하며, 팀 프로젝트의 공식 공유 상태는 중앙 Authority가 조정하는 범용 AI 개발 플랫폼.

## 핵심 문서

현재 최우선 설계는 개인 모드 MVP다.

실제 v2 구현은 별도 Public Monorepo `ai-development-platform`에서 Local Worker Golden Path부터 시작한다. 관련 결정: [[07 ADR/ADR-0013 MVP Implementation Slice and Repository Strategy]]

- [[01 Product/Product Vision]]
- [[01 Product/Personal Mode MVP]]
- [[02 Architecture/System Context]]
- [[02 Architecture/Data Ownership]]
- [[03 Domain Model/Domain Model]]
- [[04 Distributed Systems/Consistency and Messaging]]
- [[05 Database/Database Strategy]]
- [[06 Security/Identity and Approval]]
- [[07 ADR/ADR-0001 Central Authority]]
- [[07 ADR/ADR-0004 Governance and Approval Policy]]
- [[07 ADR/ADR-0005 Personal and Team Runtime Topology]]
- [[07 ADR/ADR-0006 Owner Runtime and Agent Runs]]
- [[07 ADR/ADR-0007 Autonomy and Approval Risk Policy]]
- [[07 ADR/ADR-0008 Personal Mode MVP and Deployment]]
- [[07 ADR/ADR-0009 Personal Mode Core Data Model and State Machines]]
- [[07 ADR/ADR-0010 Owner Tool Contract and Local Control Plane API]]
- [[07 ADR/ADR-0011 Personal Runtime, Account, Device, and Session Model]]
- [[07 ADR/ADR-0012 Remote Test Runner Worker Capability]]
- [[07 ADR/ADR-0013 MVP Implementation Slice and Repository Strategy]]
- [[08 Repository Audit/Keep Rewrite Delete]]
- [[09 Roadmap/Redesign Roadmap]]
- [[09 Roadmap/Personal Mode MVP Roadmap]]
- [[10 Open Questions/Open Questions]]

Real AGY Worker Alpha has been verified in controlled opt-in paths, but general user-project automation remains gated by Owner-led policy and safe pilot validation.
