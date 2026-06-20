# Domain Model

## Work Item

사람과 Owner가 보는 계획 단위. 자유로운 부모·자식 관계와 kind를 가진다.

## Task

Owner가 Worker에게 주는 실행 가능한 작은 단위.

- goal
- required_capabilities
- read_scope / write_scope / forbidden_scope
- success_criteria
- dependencies
- risk_level

## Task Attempt

Task의 실제 한 번의 실행. 재시도할 때 기존 기록을 덮어쓰지 않는다.

- attempt_id
- lease_id / lease_epoch
- assigned_node_id / assigned_worker_id
- base_commit
- status
- started_at / completed_at

## Change Package

수정되지 않는 변경 제출 기록.

- package_id
- task_attempt_id
- origin_node_id
- base_commit / head_commit
- branch_name / changed_files
- test_evidence / artifact_refs
- referenced_decisions
- signature / supersedes_package_id
