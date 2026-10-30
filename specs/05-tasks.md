# 05. Tasks (Loopship Console)

Tick a box only when the Definition of Done in `CLAUDE.md` is met. `pkg1`/`pkg2` refer to `qa-mobile-agent-spec` and `feature-implementer-spec`.

## M-1. Validate before building (do this first)

- [ ] **T-001** Run 10 to 15 interviews with mobile developers and tech leads (script in `07-business-validation.md`). Record pains, current tools, willingness to pay, and security objections.
  DoD: summary doc with quotes, counts, and a go/no-go recommendation.
- [ ] **T-002** Publish a landing page with the prototype video and a waitlist form (no tracking beyond a privacy-respecting counter).
  DoD: at least 50 qualified sign-ups or a documented reason to adjust.
- [ ] **T-003** Concierge pilot: run the open-source skills manually for 3 teams on their real tasks (their infrastructure or yours), collect results and reviewer feedback.
  DoD: per-pilot report with delivery rate, time saved (self-reported), and issues.
- [ ] **G1 Decision gate** Proceed, pivot, or stop, based on T-001 to T-003 and the thresholds in `07-business-validation.md`.

## M0. Foundation

- [ ] **T-010** Repository, TypeScript strict, ESLint/Prettier, Vitest, Playwright, CI, environments (dev, staging).
  Reqs: REQ-NFR-202. DoD: CI green; preview deploys.
- [ ] **T-011** Auth.js with GitHub OAuth; organizations, memberships, roles.
  Reqs: REQ-WEB-001, REQ-WEB-002. DoD: role-based access tests.
- [ ] **T-012** Database schema and migrations (Drizzle) for entities in `02-architecture.md`; **RLS policies** and automated cross-tenant tests.
  Reqs: REQ-WEB-081. DoD: tests prove no cross-org reads/writes for every tenant table.
- [ ] **T-013** Design system: tokens from `03-ux-and-design-system.md`, base components (Button, Input, Chip, Tabs, Table, Modal, Toast, Command palette) with accessibility tests, Storybook or equivalent.
  Reqs: REQ-NFR-203. DoD: axe checks pass; visual reference matches `prototype/`.
- [ ] **T-014** App shell: sidebar, top bar, routing, project switcher, empty/loading/error patterns.
  DoD: keyboard navigation and focus order verified.

## M1. Core run pipeline (MVP slice: Android, Jira + Figma + GitHub, one hosted runner)

- [ ] **T-020** **Spike:** run the skills with the chosen agent engine in headless/programmatic mode on a Linux box against the fixture app; record reliability, cost, and time. Decide the engine (OQ-203) in an ADR.
  DoD: ADR + a reproducible script.
- [ ] **T-021** `runner-agent`: register, heartbeat, lease, ephemeral workspace, supervise agent process, call `loop decide` and `guard check` between steps, stream events, upload artifacts, wipe workspace.
  Reqs: REQ-WEB-036, REQ-WEB-060, REQ-WEB-083. DoD: simulated-run tests; workspace wipe verified.
- [ ] **T-022** Runner gateway (WebSocket) and job queue (`pg-boss`) with leases, retries, and cancellation.
  Reqs: REQ-WEB-031, REQ-WEB-032. DoD: lease-expiry and cancel tests.
- [ ] **T-023** Runs API and SSE event stream with persistence and replay (`Last-Event-ID`).
  Reqs: REQ-WEB-030, REQ-WEB-040. DoD: contract tests; reconnect test.
- [ ] **T-024** GitHub App: install flow, short-lived installation tokens per job, push of feature branch by the runner, **PR creation by the control plane**.
  Reqs: REQ-WEB-010, REQ-WEB-052. DoD: end-to-end on a test repo; agent cannot open PRs itself.
- [ ] **T-025** Jira and Figma integrations (OAuth, read-only), encrypted credential storage, URL parsing and provider detection.
  Reqs: REQ-WEB-020..022, REQ-WEB-080. DoD: token never reaches the browser; encryption tests.
- [ ] **T-026** Artifact storage: pre-signed uploads/downloads, per-org prefixes, content-type and size limits.
  Reqs: REQ-WEB-051, REQ-WEB-082. DoD: cross-tenant access denied in tests.
- [ ] **T-027** Hosted Android runner image (emulator with hardware acceleration, SDK, skills, CLI tools) and deployment to a KVM-capable host.
  Reqs: REQ-WEB-060. DoD: a fixture run completes on the hosted runner.

## M2. UI

- [ ] **T-030** Runs list with filters, search, status chips.
- [ ] **T-031** New run form with provider detection chips, mode control, limits, and validation errors.
- [ ] **T-032** Run detail: header, pipeline stepper, timeline, log viewer (virtualized, follow mode, redaction).
- [ ] **T-033** Device mirror: frame channel, adaptive frame rate, tap/swipe overlays synchronized with events, disconnected/stale states.
  Reqs: REQ-WEB-041, REQ-WEB-042. DoD: latency measured against REQ-NFR-201.
- [ ] **T-034** Result view: verdict banner, criteria, bugs with evidence, regression table, design comparison slider, cycle history, evidence gallery with video.
  Reqs: REQ-WEB-050..055. DoD: renders all example verdicts in `contracts/examples/`.
- [ ] **T-035** Approvals (modal, audit log) and PR actions ("Approve and open PR", "Request changes").
  Reqs: REQ-WEB-033.
- [ ] **T-036** Projects and Integrations screens including the "what this can access" disclosure.
- [ ] **T-037** Runners screen (hosted pool status, health).

  DoD for all M2 tasks: loading/empty/error/permission states, keyboard operation, WCAG AA checks, Playwright happy-path test.

## M3. Cost control and safety

- [ ] **T-040** Budgets (cycles, minutes, spend), metering, org and per-run caps with hard stop.
  Reqs: REQ-WEB-034, REQ-WEB-070, REQ-WEB-072. DoD: run is stopped when the cap is reached (test).
- [ ] **T-041** Stripe plans and usage-based billing (behind a feature flag until pricing is validated).
  Reqs: REQ-WEB-071.
- [ ] **T-042** Retention and deletion jobs; "delete my data" flow.
  Reqs: REQ-WEB-082.
- [ ] **T-043** Audit log for sensitive actions.
  Reqs: REQ-WEB-003.
- [ ] **T-044** Hardening: egress allowlist on runners, secret redaction in the event pipeline, safe rendering of untrusted content, prominent prompt-injection findings.
  Reqs: REQ-WEB-043, REQ-WEB-055, REQ-WEB-083. DoD: security tests in `06-testing-ops-security.md` pass.
- [ ] **T-045** Observability and alerts (metrics, traces, stuck-lease and cost-anomaly alerts).

## M4. Self-hosted runners and API

- [ ] **T-050** Self-hosted runner package (container image + install command), outbound-only, one-time registration token, revocation.
  Reqs: REQ-WEB-061, REQ-WEB-063.
- [ ] **T-051** Public API with API keys and scopes; outbound webhooks with signatures and delivery log.
  Reqs: REQ-WEB-090, REQ-WEB-091.
- [ ] **T-052** Documentation site: quick start, security model, runner setup, API reference.

## M5. Private beta

- [ ] **T-060** Onboarding with a sample project and a guided first run.
- [ ] **T-061** Beta with 3 to 5 teams; weekly review of runs, escalations, and costs.
- [ ] **T-062** Reliability fixes from beta findings (top 5 escalation reasons).
- [ ] **T-063** Pricing experiments (see `07-business-validation.md`).
- [ ] **G2 Decision gate** Expand, adjust, or stop, based on delivery rate, margin, and retention.

## M6. Later

- [ ] **T-070** iOS runners: spike on Mac hosts, licensing, simulator lifecycle (Reqs: REQ-WEB-062).
- [ ] **T-071** iOS mirror and evidence pipeline.
- [ ] **T-072** Trello and Azure DevOps connectors (REQ-WEB-023).
- [ ] **T-073** H.264 mirror for Android (smoother streaming).
- [ ] **T-074** SSO/SAML, data residency options, SOC 2 readiness.

## Definition of Ready

Requirement IDs listed; dependencies ticked; relevant open questions (`OQ-2xx`) answered or deferred in the PR description.
