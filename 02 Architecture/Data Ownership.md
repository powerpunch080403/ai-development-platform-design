# Data Ownership

| 데이터 | 권위 있는 소유자 |
|---|---|
| Workspace, 멤버십, 프로젝트 권한 | 중앙 Authority |
| 공식 Work Item과 Task | 중앙 Authority |
| Lease와 Lock | 중앙 Authority |
| Merge Queue | 중앙 Authority |
| 공식 프로젝트 결정 | 중앙 Authority |
| 사용자와 Owner의 원본 대화 | 개인 Node |
| 개인 선호와 개인 메모리 | 개인 Node |
| 로컬 실행 로그 | 개인 Node |
| 코드와 Commit 이력 | Git Remote |
| 승인된 Change Package와 감사 기록 | 중앙 Authority |

## 금지

- 개인 Node DB와 중앙 DB의 양방향 테이블 복제
- 여러 개인 Node가 같은 공유 상태의 동등한 Writer가 되는 구조
- Timestamp만으로 충돌을 덮어쓰는 Last Write Wins
