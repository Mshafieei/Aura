# Agent Factory and Skill Factory — Technical Specification

## Shared lifecycle
`draft → validating → testing → review_required → approved → staged → published → paused/retired`
Failed validation or tests must not transition to published. Published versions are immutable; fixes create a new version.

## AgentSpec (minimum fields)
- schema_version, agent_id, name, description, owner_id
- objective, triggers, input_schema, output_schema
- allowed_skill_refs pinned to exact skill versions
- permissions (least privilege, explicit scopes)
- model_policy, tool_allowlist
- limits: max_steps, max_delegation_depth, timeout_seconds, max_cost, max_tokens
- approval_policy, failure_policy, evaluation_suite_ref
- version, changelog, created_at, created_by, lifecycle_state

## SkillSpec (minimum fields)
- schema_version, skill_id, name, description, owner_id, version
- implementation_kind (deterministic, model-assisted, connector)
- input_schema, output_schema
- permissions, dependencies pinned to versions
- timeout, retry policy, idempotency behavior, rate limit
- error contract, test cases, evaluation report, changelog
- artifact digest, sandbox profile, lifecycle state

## Factory pipeline
1. Capture intent and ask only essential clarifying questions.
2. Produce a draft spec and explain capabilities/limitations.
3. Validate schema, names, dependency graph, permissions, quotas, and policy.
4. Generate implementation artifact in isolated workspace.
5. Run formatter/static analysis, unit/contract tests, adversarial tests, and resource-limit tests.
6. Produce review report including tests, permission diff, cost estimates, and known limitations.
7. Require owner approval for publish and each new permission scope.
8. Publish immutable version to staged channel; smoke-test; promote only after approval.
9. Track executions and errors; pause on configured safety thresholds.
10. Roll back to a prior immutable version without rewriting history.

## Runtime invariants
- The runtime accepts only published, enabled versions.
- Each execution pins agent and skill versions.
- No dynamic eval or unrestricted shell in the production API process.
- Generated code runs in a disposable sandbox with CPU/RAM/time limits and default-deny network egress.
- A tool result is not a permission grant.
- Untrusted content cannot alter system policies, allowlists, or approval rules.
- Agent delegation is bounded; cycles and recursive spawning are rejected.
- Human approval is a signed, time-bound decision tied to exact action payload and version.
- External side effects are idempotent where possible and have explicit outcome states.

## API outline
- POST /v1/factory/skills/drafts
- POST /v1/factory/skills/{id}/validate
- POST /v1/factory/skills/{id}/test
- POST /v1/factory/skills/{id}/request-approval
- POST /v1/skills/{id}/publish
- POST /v1/skills/{id}/pause
- POST /v1/skills/{id}/rollback
- POST /v1/factory/agents/drafts
- POST /v1/factory/agents/{id}/validate
- POST /v1/factory/agents/{id}/simulate
- POST /v1/factory/agents/{id}/request-approval
- POST /v1/agents/{id}/publish
- POST /v1/agents/{id}/pause
- POST /v1/agents/{id}/rollback
- GET /v1/agents, GET /v1/skills, GET /v1/executions/{id}, GET /v1/approvals/{id}

All endpoints require authenticated owner-scoped access, validation, audit events, and rate limits. Publishing endpoints must verify a current approval bound to the exact artifact digest, version, permissions, and environment.

## Required tests
- invalid schema and undeclared permission rejected
- dependency cycle rejected
- unpublished skill cannot run
- changed artifact digest invalidates prior approval
- prompt injection in retrieved document cannot add tools or permissions
- recursive agent creation hits depth/step limits
- sandbox timeout kills the run
- publish without approval returns forbidden
- rollback restores previous immutable version
- cross-user access is denied
- duplicate side-effect request is idempotent
