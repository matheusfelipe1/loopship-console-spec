# 04. API and events

Base path `/api/v1`. JSON over HTTPS. Errors use `application/problem+json` (RFC 9457) with `type`, `title`, `status`, `detail`, and a stable `code`.

## 1. Authentication

- Browser: session cookie (Auth.js), CSRF protection on mutating requests.
- Programmatic: `Authorization: Bearer <api_key>`; keys are hashed at rest, scoped (`runs:read`, `runs:write`, `projects:read`), and shown once.
- Runner gateway: runner credential issued at registration (see section 5).

## 2. REST endpoints

### Runs

| Method | Path | Description |
|---|---|---|
| POST | `/runs` | Create a run |
| GET | `/runs` | List runs (filters: `project_id`, `status`, `q`, cursor pagination) |
| GET | `/runs/{id}` | Run summary (status, step, cycle, verdict, cost, pr_url) |
| POST | `/runs/{id}/cancel` | Cancel |
| POST | `/runs/{id}/resume` | Resume from persisted state |
| GET | `/runs/{id}/events` | **SSE** stream; supports `Last-Event-ID` to replay from a sequence number |
| GET | `/runs/{id}/cycles` | Cycle history |
| GET | `/runs/{id}/verdict?cycle=N` | `verdict.json` (schema-validated) |
| GET | `/runs/{id}/report` | `implementation-report.json` |
| GET | `/runs/{id}/artifacts` | List artifacts with short-lived signed URLs |
| POST | `/runs/{id}/approvals/{approval_id}` | Decide an approval: `{ "decision": "approve" \| "reject" \| "answer", "answer": "..." }` |
| POST | `/runs/{id}/share` | Create an expiring, revocable share link |

`POST /runs` body:

```json
{
  "project_id": "prj_123",
  "task_url": "https://example.atlassian.net/browse/PROJ-123",
  "design_url": "https://www.figma.com/design/EXAMPLEKEY/Signup?node-id=12-40",
  "platform": "android",
  "target": "emulator",
  "mode": "implement_and_validate",
  "budget": { "max_cycles": 3, "max_minutes": 90, "max_cost_cents": 1500 }
}
```

Validation: URLs parsed and provider-detected; unsupported providers return `422` with `code: "unsupported_provider"`; budgets cannot exceed org caps.

### Projects, integrations, runners, org

| Method | Path | Description |
|---|---|---|
| GET/POST | `/projects` | List/create projects |
| GET/PATCH/DELETE | `/projects/{id}` | Read/update/delete; config validated against `config.schema.json` |
| POST | `/projects/{id}/validate-config` | Returns schema errors with JSON pointers |
| GET | `/integrations` | List with connection state and scopes |
| POST | `/integrations/{type}/connect` | Start OAuth; returns redirect URL |
| DELETE | `/integrations/{id}` | Disconnect and delete credentials |
| GET | `/runners` | Hosted pool status and self-hosted runners |
| POST | `/runners/registration-tokens` | Create a one-time token for a self-hosted runner |
| DELETE | `/runners/{id}` | Revoke |
| GET | `/usage` | Usage and spend by period |
| GET/POST/DELETE | `/api-keys` | Manage API keys |
| GET | `/audit-log` | Paginated audit events |
| GET/PATCH | `/org` | Org settings (retention, spend caps) |

### Webhooks (outbound)

`run.completed`, `run.escalated`, `approval.requested`. Signed with HMAC-SHA256 (`X-Loopship-Signature`), retried with exponential backoff, delivery log visible in the UI.

## 3. Event stream

Each event: `{ "seq": 128, "ts": "2026-10-29T15:20:11.430Z", "type": "...", "payload": { ... } }`. SSE `id` is `seq`. Events are persisted in `run_events` and replayable.

| Type | Payload (key fields) |
|---|---|
| `run.created` / `run.started` | `run_id`, `runner_id` |
| `step.started` / `step.completed` | `step`, `cycle`, `duration_ms` |
| `log.line` | `stream` (`agent`, `build`, `device`), `text` (redacted), `level` |
| `action.performed` | `kind` (`tap`, `type`, `swipe`, `back`, `launch`), `x`, `y` (0 to 1), `label`, `step_id` |
| `frame.captured` | `frame_id`, `seq`, `width`, `height` (the image itself travels on the mirror channel) |
| `criterion.updated` | `id`, `status` |
| `bug.found` | `bug_id`, `severity`, `title`, `regression`, `evidence` |
| `design.compared` | `screen`, `status`, `side_by_side_artifact_id` |
| `guard.finding` | `rule`, `severity`, `path`, `detail` |
| `verdict.issued` | `cycle`, `verdict`, `reasons`, `verdict_artifact_id` |
| `decision.made` | `action`, `reason`, `bugs_to_fix` (output of `loop decide`) |
| `cycle.completed` | `cycle`, `files_changed`, `duration_s` |
| `approval.requested` / `approval.resolved` | `approval_id`, `kind`, `prompt`, `options` / `decision` |
| `security.finding` | `kind` (`possible_prompt_injection`), `source`, `excerpt` |
| `pr.opened` | `pr_url`, `branch` |
| `budget.updated` | `minutes_used`, `cost_cents`, `cycles_used` |
| `run.completed` / `run.failed` | `final_status`, `escalation_reason?` |

## 4. Mirror channel

`GET /runs/{id}/mirror` upgrades to WebSocket (auth required; org-scoped). Server sends binary messages: a small JSON header (`frame_id`, `seq`, `ts`, `width`, `height`, `mime`) followed by the image bytes. The client may send `{ "type": "subscribe", "max_fps": 2 }` to cap bandwidth. Frames are not persisted by default (only screenshots that are evidence).

## 5. Runner gateway protocol (outbound-only)

1. **Register:** `POST /runner/v1/register` with a one-time token and capabilities (`platforms`, `capacity`, `version`). Response: runner id and a long-lived credential (rotatable).
2. **Connect:** WebSocket `/runner/v1/connect` with the credential. Heartbeat every 15 s. Server closes stale connections after 45 s.
3. **Lease:** server sends `job.assigned`:

```json
{
  "type": "job.assigned",
  "job_id": "job_9",
  "run_id": "run_42",
  "lease_expires_at": "2026-10-29T16:20:00Z",
  "spec": {
    "mode": "implement_and_validate",
    "repo": { "clone_url": "https://github.com/example/app.git", "base_branch": "main", "token_ref": "short-lived" },
    "task_url": "...", "design_url": "...",
    "config": { "...": "validated against config.schema.json" },
    "skills": { "mobile-qa": "0.1.0", "feature-implementer": "0.1.0", "cli": "0.1.0" },
    "budget": { "max_cycles": 3, "max_minutes": 90, "max_cost_cents": 1500 },
    "secrets": "delivered as ephemeral env, not in this payload"
  }
}
```

4. **Report:** runner sends `event` messages (same types as the stream), `artifact.upload_request` (receives pre-signed URL), `lease.extend`, `job.completed` (with `implementation-report.json`) or `job.failed`.
5. **Control messages from server:** `job.cancel`, `approval.resolved`, `config.reload`.
6. Runners never accept inbound connections.

## 6. Rate limits and quotas

Per API key and per org: request rate limits with `429` and `Retry-After`; concurrent run limits by plan; artifact size limits (screenshots 10 MB, videos 500 MB per file default).

## 7. Versioning

`/api/v1` is stable; additive changes only. Contracts embedded in payloads carry their own `schema_version`.
