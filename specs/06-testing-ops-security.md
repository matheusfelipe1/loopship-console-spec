# 06. Testing, operations, and security

## 1. Testing

| Layer | Scope | Tools |
|---|---|---|
| Unit | Business logic, URL parsing, budget math, event reducers | Vitest |
| Integration | API + DB with RLS, queue/leases, artifact signing, integrations with fakes | Vitest, Testcontainers |
| Contract | Every JSON file crossing a boundary validates against `contracts/` | Ajv in tests |
| End to end | Critical UI paths with a simulated runner replaying recorded event streams | Playwright |
| Runner simulation | A fake runner that emits scripted events/frames to test the live view, reconnects, and failure states | Custom harness |
| Real runs | Hosted runner against the fixture app (nightly) | Scripted |
| Load | Concurrent SSE/WebSocket connections, event bursts, queue throughput | k6 or similar |
| Chaos | Kill runners mid-job, drop connections, expire leases, DB failover drills | Scripted |
| Accessibility | axe in CI + manual keyboard/screen-reader pass for core flows | axe, Playwright |
| Visual regression | Key screens in dark theme | Playwright screenshots |

Mandatory automated tests: **cross-tenant isolation** for every tenant table, endpoint, artifact URL, SSE channel, and mirror channel; budget hard stops; cancellation cleanup; secret redaction; webhook signatures.

## 2. Operations

- **Environments:** local, staging, production; infrastructure as code; secrets via a managed secret store.
- **CI/CD:** lint, type-check, tests, security scans (dependencies, containers); migrations run with backups and are backward compatible; preview deployments.
- **Backups and recovery:** automated Postgres backups with point-in-time recovery; restore drills quarterly; object storage versioning for reports.
- **SLOs (MVP):** control plane availability 99.5%; event latency p95 < 1 s; run start latency p95 < 2 min when capacity exists.
- **Runbooks:** stuck lease, runner fleet unhealthy, queue backlog, cost spike, suspected data exposure.
- **Capacity:** track emulator start time, runner utilization, and cost per run; maintain a warm pool when justified by data.
- **Support:** in-app feedback, status page, response targets for beta customers.

## 3. Threat model (hosted product)

| # | Threat | Impact | Mitigations |
|---|---|---|---|
| H1 | Cross-tenant data access (IDOR, RLS gaps) | Severe | RLS + tests for each endpoint/channel; opaque ids; signed, short-lived URLs; per-org storage prefixes |
| H2 | Malicious repository or build scripts attack our runner infrastructure | Severe | One job per container/VM, no shared writable volumes, no privileged mode, egress allowlist, resource limits, no metadata-service access, wipe on exit |
| H3 | Prompt injection in tickets/designs/app text leads the agent to exfiltrate secrets or take unwanted actions | High | Untrusted-content rules in skills; egress allowlist; ephemeral, scoped credentials; agent cannot open PRs or merge; findings surfaced in UI |
| H4 | Token theft (integration credentials, GitHub installation tokens) | High | Envelope encryption with KMS; server-side refresh; short-lived job tokens; never sent to the browser; rotate on disconnect |
| H5 | SSRF via user-supplied URLs (task/design links) | High | Provider allowlist and strict URL parsing; resolve and block private ranges; fetch only through integration clients |
| H6 | Stored XSS from agent output, tickets, logs | High | Escape everything; sanitize Markdown; CSP; images from a separate origin; no inline scripts |
| H7 | Cost abuse (loops, crypto-mining, spam runs) | Medium to High | Budgets, org caps, per-run hard stops, rate limits, anomaly alerts, runner CPU/network policies |
| H8 | Evidence contains personal data and is exposed | Medium to High | Private by default, expiring share links, retention policies, redaction, user warnings |
| H9 | Supply chain (skills, CLI, dependencies, runner images) | High | Pinned and signed releases, provenance, SBOM, image scanning, minimal base images |
| H10 | Insider access | High | Least-privilege operations access, audit logging, break-glass procedure |
| H11 | GitHub App over-permission | Medium | Minimal scopes, per-installation tokens, docs recommending branch protection |
| H12 | Replay/forgery of webhooks | Medium | HMAC signatures with timestamps, replay window |

## 4. Privacy and compliance

- Data minimization: source code only on runners during jobs; do not persist clones.
- Retention: configurable per org; defaults documented; deletion on request propagates to storage and backups within a stated window.
- Documents to prepare before beta: Terms of Service, Privacy Policy, Data Processing Agreement, subprocessor list, security overview page.
- Regulations to consider: GDPR (EU) and LGPD (Brazil) for personal data in evidence; export controls and sanctions screening for sign-ups if applicable (confirm with counsel).
- Apple macOS licensing for iOS runners: verify terms before building.
- Model provider terms: confirm usage terms for hosted agent workloads and customer data handling.

## 5. Incident response

Severity levels, on-call rota (even if one person), a 72-hour customer notification target for confirmed data exposure (align with counsel), post-incident review template, and a public status page.

## 6. Pre-launch checklist

- [ ] Cross-tenant test suite green
- [ ] Secrets scan and dependency scan clean
- [ ] Egress allowlist verified from a runner
- [ ] Injection fixtures verified end to end
- [ ] Budget caps verified with real metering
- [ ] Backup restore drill done
- [ ] Legal documents reviewed by a professional
- [ ] Status page and support channel live
