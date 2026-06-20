---
type: adr
status: accepted
date: 2026-06-20
---

# ADR-0001: 공유 프로젝트 상태는 중앙 Authority가 관리한다

## Context

여러 사용자가 각자 개인 서버와 Owner를 가지고 하나의 프로젝트에 협업한다.

## Decision

공유 프로젝트 상태는 프로젝트별 중앙 Authority 서버가 단독으로 관리한다. 개인 Node는 중앙 상태의 동등한 복제본이나 Writer가 아니다.

## Consequences

- 멀티 마스터 DB 충돌을 피한다.
- 오프라인 개인 작업은 가능하지만 공식 상태 변경은 보류된다.
- Authority 장애 시 공식 병합과 새 작업 배정은 중단된다.
