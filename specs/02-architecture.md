# 02. Architecture

## 1. Overview

```
 Browser (Next.js UI) <----- SSE events / WebSocket mirror -----+
        |  REST                                                  |
        v                                                        |
 +--------------------------- Control plane ------------------------------+
 |  API (route handlers)  | Orchestrator (pg-boss)  | Event hub            |
 |  Integrations (Jira/Figma/GitHub) | Billing (Stripe) | Artifact signer  |
 |  Auth.js (GitHub OAuth) | Audit log | Runner gateway (WebSocket)        |
 +---------+---------------------------+------------------------+---------+
           |                           |                        |
      PostgreSQL                 Object storage            Runner gateway
   (RLS by org_id)          (artifacts, evidence)                |
                                                                 | outbound-only
                                              +------------------+-----------------+
                                              |                                    |
                                     Hosted runner (Linux)               Self-hosted runner
                                     Docker, emulator (KVM)              customer network
                                     runner-agent + agent engine         same runner-agent
                                     + skills + CLI tools + device tools
```

The runner executes the open-source skills. The control plane never runs customer code.

## 2. Stack decisions (ADR 0001)

| Concern | Choice | Reason |
|---|---|---|
| Language | TypeScript end to end | One language for UI, API, runner agent, and CLI tools |
| Web framework | Next.js (App Router) | UI + API in one deployable for the MVP |
| Styling | Tailwind CSS + design tokens; Radix UI primitives | Accessible primitives, dark-theme friendly |
| Database | PostgreSQL 16 + Drizzle ORM; Row-Level Security by `org_id` | Strong tenancy guarantees, simple ops |
| Queue | `pg-boss` (Postgres-based) | No extra infrastructure for MVP; revisit at scale |
| Realtime | SSE for events; WebSocket for mirror frames and runner gateway | SSE is simple and proxy-friendly; frames need binary |
| Storage | S3-compatible (S3, R2, or MinIO) with pre-signed URLs | Evidence can be large; signed access per tenant |
| Auth | Auth.js with GitHub OAuth | Matches target users; SSO later |
| Billing | Stripe | Standard |
| Agent engine on runners | Claude Code in programmatic/headless mode or the Claude Agent SDK (decided by spike T-020) | Skills are portable; the engine is replaceable |
| Tests | Vitest, Playwright | |

## 3. Core entities

| Entity | Key fields |
|---|---|
| `organizations` | id, name, plan, monthly_spend_cap, retention_days |
| `users`, `memberships` | user id, org id, role (`owner`, `admin`, `member`, `viewer`) |
| `projects` | id, org_id, repo (provider, owner, name), base_branch, config (JSON, validated by `config.schema.json`), platforms, targets |
| `integrations` | id, org_id, type (`jira`, `figma`, `github`, later `trello`, `azure_devops`), status, scopes, credentials_ref (envelope-encrypted), created_by |
| `runs` | id, org_id, project_id, created_by, mode, task_url, design_url, issue_key, platform, target, status, step, cycle, max_cycles, budget (JSON), final_status, verdict, pr_url, escalation_reason, cost_cents, started_at, finished_at |
| `run_cycles` | run_id, cycle, verdict, bugs_fixed, bugs_open, files_changed, duration_s, verdict_artifact_id |
| `run_events` | run_id, seq, ts, type, payload (JSON) (append-only) |
| `bugs` | run_id, cycle, bug_id, fingerprint, severity, title, regression, evidence refs (queryable copy of verdict bugs) |
| `artifacts` | id, run_id, cycle, kind (`screenshot`, `video`, `log`, `report`, `verdict`, `comparison`), storage_key, size, content_type, sha256, expires_at |
| `approvals` | id, run_id, kind (`clarification`, `open_pr`, `accept_caveats`), prompt, options, status, decided_by, decided_at |
| `runners` | id, org_id (null for hosted pool), name, kind (`hosted`, `self_hosted`), platforms, capacity, version, status, last_seen_at |
| `api_keys` | id, org_id, hashed_key, scopes, last_used_at |
| `usage_records` | org_id, run_id, kind (`runner_minutes`, `agent_cost`), quantity, cost_cents, ts |
| `audit_log` | org_id, actor, action, target, ts, metadata |

All tenant tables carry `org_id` and are protected by RLS policies. `runs.status` is one of `queued`, `running`, `succeeded`, `escalated`, `failed_environment`, `cancelled`, `failed`. `runs.step` is one of `reading`, `planning`, `implementing`, `verifying`, `building`, `validating`, `deciding`, `fixing`, `delivering`.

Mapping from `implementation-report.final_status`: `pr_opened` and `go_no_pr` map to `succeeded`; `escalated`, `failed_environment`, and `cancelled` keep their names.

## 4. Run lifecycle

1. **Create:** API validates URLs, project, budgets; writes `runs` row (`queued`); enqueues a job.
2. **Lease:** an eligible runner (platform, capacity) leases the job over the gateway with a time-bound lease.
3. **Prepare:** runner creates an ephemeral workspace; clones the repo with a short-lived GitHub installation token; installs the pinned skills and CLI tools; receives ephemeral secrets (agent API key, tracker/design tokens) as environment variables, never written to disk.
4. **Execute:** runner starts the agent engine with the skill (`feature-implementer` or `mobile-qa`). Between agent steps the **runner-agent calls `loop decide` and `guard check`** and enforces the result (REQ-WEB-036).
5. **Stream:** runner emits events (04) and captures mirror frames; uploads artifacts to pre-signed URLs.
6. **Approvals:** when an `approval.requested` event appears, the run pauses (`running`, waiting flag) until a human decides in the UI.
7. **Deliver:** runner pushes the feature branch. After approval (or per policy), the **control plane** opens the PR through the GitHub App, so the agent never holds PR-opening capability.
8. **Finish:** runner uploads the final report, wipes the workspace, and releases the lease. Control plane updates final status, usage, and notifies via webhook.

Failure handling: lease expiry returns the job to the queue (up to N retries); environment failures follow the `ENVIRONMENT_FAILURE` rules of the loop; cancellation sends a cancel signal; runners wipe workspaces on every exit path.

## 5. Runners

| Kind | Platform | Notes |
|---|---|---|
| Hosted Android (MVP) | Linux | Android emulator needs hardware virtualization (KVM). Use bare-metal or VMs with nested virtualization. |
| Hosted iOS (v1.2) | macOS on Apple hardware | iOS simulators require macOS and Xcode; Apple's license terms generally require macOS to run on Apple-branded hardware, so plan for Mac hosts or a provider that offers them. Verify terms before launch. |
| Self-hosted | Any | Same `runner-agent`, outbound-only connection, registered with a one-time token; customer controls hardware and network. |

Runner agent responsibilities: register, heartbeat (15 s), lease jobs, prepare workspace, supervise the agent process, enforce time and spend budgets, call deterministic tools, stream events, upload artifacts, wipe workspace.

Isolation (ADR 0003): one job per container/VM; no shared writable volumes; network egress allowlist (package registries, the code host, tracker/design APIs, the model API, object storage); resource limits; no inbound ports.

## 6. Device mirror

**MVP (v1):** the runner captures a screenshot every 500 to 1000 ms with `adb exec-out screencap -p` (Android) or `xcrun simctl io booted screenshot` (iOS), downscales and encodes (WebP/JPEG), and sends frames over the runner WebSocket tagged with a sequence number and timestamp. The control plane relays frames to subscribed browsers over a WebSocket. Frame rate adapts to bandwidth. Action overlays come from `action.performed` events (coordinates normalized 0 to 1) so the UI can draw tap markers without touching the frame.

**v2:** for Android, stream H.264 from the scrcpy server to the browser (WebCodecs or MSE) for smoother mirroring. iOS near-real-time streaming is harder; keep screenshot polling and consider a window-capture pipeline on Mac hosts.

Honest limit: the mirror is for following the run, not for interactive control. Interactive "take over" is out of scope for v1.

## 7. Integrations

| Integration | Access | Notes |
|---|---|---|
| GitHub (App) | `metadata:read`, `contents:write`, `pull_requests:write` on selected repositories; installation tokens issued per job (short-lived) | Recommend branch protection/rulesets on base; the agent cannot merge |
| Jira (Atlassian OAuth) | Read issues by default; write (create bugs, comment) is opt-in | Cloud only via the official server; Data Center via a community server is a separate evaluation |
| Figma | Read-only OAuth/token | Link-based access |
| Trello, Azure DevOps (v1.1) | Via vendor APIs or MCP servers if available | Verify availability during implementation |

Credentials are envelope-encrypted (data key per integration, wrapped by a KMS key). Tokens are refreshed server-side and handed to jobs as short-lived values.

## 8. Security architecture (summary; full list in 06)

- Tenant isolation: RLS on all tenant tables + tests; per-org object prefixes; signed URLs with short TTL.
- Secrets: KMS envelope encryption; never logged; redaction in event pipeline.
- Jobs: ephemeral workspaces, egress allowlist, no inbound access.
- Agent output and external text rendered as escaped text; Markdown sanitized; images served from a separate origin.
- Audit log for sensitive actions.

## 9. Observability

Structured logs with `run_id`/`org_id`; OpenTelemetry traces across API, queue, and gateway; metrics: queue depth, lease latency, runner utilization, run duration, escalation reasons, cost per run, frame latency; alerting on stuck leases, rising failure rate, and spend anomalies.

## 10. Scaling notes (not for MVP)

Swap `pg-boss` for a dedicated queue and add Redis/NATS for event fan-out when SSE connections or queue throughput outgrow Postgres. Shard runners by platform and region. Introduce warm emulator pools to cut start time.

## 11. Open questions

- **OQ-201** Hosting provider for the control plane and for KVM-capable Android runners.
- **OQ-202** Managed model key vs bring-your-own-key at launch (MVP assumes bring-your-own-key to bound cost and risk).
- **OQ-203** Agent engine choice (spike T-020).
- **OQ-204** Where evidence lives for self-hosted runners (customer bucket vs ours).
- **OQ-205** Data residency and compliance roadmap (SOC 2, GDPR/LGPD).
