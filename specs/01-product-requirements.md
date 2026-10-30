# 01. Product requirements (PRD)

## 1. Vision

Turn "here is the ticket and the design" into "here is a reviewed, tested pull request with evidence", with a human making the final call.

## 2. Personas

| Persona | Needs | Pain today |
|---|---|---|
| **Mobile developer** (primary) | Offload small, well-specified tasks; get evidence-backed PRs | Context switching, manual testing on devices, design drift |
| **Tech lead / engineering manager** | Throughput without quality loss; auditability | Review burden, inconsistent testing, no proof of what was tested |
| **QA engineer** | Automated validation against criteria and design; regression upkeep | Manual repetitive checks, flaky scripts |
| **Product designer** | Confidence that implementation matches the design | Finding divergences late |

Initial target customer (hypothesis): mobile teams of 5 to 50 engineers without a dedicated QA automation team, using Jira + Figma + GitHub. To be validated (`07-business-validation.md`).

## 3. Jobs to be done

- When I have a clear ticket and a design, I want a first implementation tested on a device so that I review a PR instead of starting from zero.
- When a feature is ready, I want an independent validation with evidence so that I can decide to ship with confidence.
- When something fails, I want to see exactly what happened (video, screenshots, logs) so that I can intervene quickly.

## 4. Core user stories

1. As a developer, I paste a Jira link and a Figma link, choose Android and Run, and watch progress live.
2. As a developer, I see the app running in a device mirror with tap markers and the current step.
3. As a reviewer, I open a finished run and see criteria status, bugs with evidence, regression results, Figma vs real comparisons, and the verdict.
4. As a tech lead, I set limits (max cycles, spend) and see every run's audit trail.
5. As a developer, when a run escalates, I read why, answer a clarifying question, and resume or re-run.
6. As an admin, I connect Jira, Figma, and GitHub with minimal permissions and see what the product can access.
7. As a security-conscious customer, I run jobs on my own runner so source code never leaves my network.

## 5. Functional requirements

### Accounts and organizations
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-001 | MUST | Sign in with GitHub OAuth. |
| REQ-WEB-002 | MUST | Organizations with roles: owner, admin, member, viewer. |
| REQ-WEB-003 | MUST | Audit log of security-relevant actions (connect integration, create run, approve PR, change limits). |

### Projects and integrations
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-010 | MUST | Create a project by linking a GitHub repository (GitHub App with minimal permissions) and setting base branch, lint/test/build commands, targets, and limits (maps to `config.schema.json`). |
| REQ-WEB-011 | SHOULD | Detect the stack (Android, iOS, Flutter, React Native) and suggest default commands. |
| REQ-WEB-012 | MUST | Validate project configuration against the shared schema and show actionable errors. |
| REQ-WEB-020 | MUST | Connect Jira (OAuth) with read access; write access is a separate, opt-in permission. |
| REQ-WEB-021 | MUST | Connect Figma (OAuth or token) read-only. |
| REQ-WEB-022 | MUST | Parse task and design URLs: recognize Jira, Trello, Azure DevOps, Figma, and show a clear message for unsupported ones (for example Adobe XD). |
| REQ-WEB-023 | SHOULD | Trello and Azure DevOps connectors (v1.1). |

### Runs
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-030 | MUST | Create a run from task URL, optional design URL, project, platform, target, and mode (`implement_and_validate` or `validate_only`). |
| REQ-WEB-031 | MUST | Runs are queued and executed on a runner; status model: queued, running (with step), succeeded, escalated, failed_environment, cancelled, failed. |
| REQ-WEB-032 | MUST | Cancel a run at any time; the runner stops and cleans up. |
| REQ-WEB-033 | MUST | Approvals: the run can pause for a human decision (clarify criteria, open PR, accept caveats). |
| REQ-WEB-034 | MUST | Per-run budgets (max cycles, max minutes, max spend) with visible consumption. |
| REQ-WEB-035 | SHOULD | Re-run and resume from persisted state (`run-state.json`). |
| REQ-WEB-036 | MUST | The console invokes `loop decide` and `guard check` between agent steps (via the runner), not only the agent. |

### Live view
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-040 | MUST | Live pipeline and timeline via an event stream. |
| REQ-WEB-041 | MUST | Device mirror: near-real-time frames of the device screen. |
| REQ-WEB-042 | SHOULD | Action overlay on the mirror (tap/swipe/type markers) synchronized with the timeline. |
| REQ-WEB-043 | MUST | Log viewer with secret redaction, search, and follow mode. |
| REQ-WEB-044 | SHOULD | Pause and resume at safe points. |

### Results
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-050 | MUST | Result view: verdict banner, criteria table, bugs with severity and evidence, regression table, design comparison, cycle history. |
| REQ-WEB-051 | MUST | Evidence gallery with video playback and screenshot zoom; downloads. |
| REQ-WEB-052 | MUST | Show PR link; export report as Markdown and PDF. |
| REQ-WEB-053 | SHOULD | Compare cycles side by side. |
| REQ-WEB-054 | SHOULD | Share links are private by default, expiring, and revocable. |
| REQ-WEB-055 | MUST | Display findings of possible prompt injection prominently. |

### Runners
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-060 | MUST | Hosted Android runners (Linux, emulator). |
| REQ-WEB-061 | SHOULD | Self-hosted runners via an outbound-only agent registered with a one-time token. |
| REQ-WEB-062 | SHOULD | Hosted iOS runners on Apple hardware (v1.2). |
| REQ-WEB-063 | MUST | Runner health, capacity, and version visible; stale runners are removed from scheduling. |

### Billing and limits
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-070 | MUST | Meter usage: runner minutes and agent cost per run. |
| REQ-WEB-071 | SHOULD | Plans via Stripe: free tier and team plan (hypotheses in `07-business-validation.md`). |
| REQ-WEB-072 | MUST | Org-level monthly spend cap and per-run cap; hard stop when reached. |

### Security and compliance
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-080 | MUST | Integration credentials encrypted at rest with envelope encryption; never exposed to the browser. |
| REQ-WEB-081 | MUST | Tenant isolation verified by automated tests (no cross-org reads of runs, artifacts, integrations). |
| REQ-WEB-082 | MUST | Configurable retention for artifacts and logs; deletion on request; source code never persisted after job end. |
| REQ-WEB-083 | MUST | Ephemeral job workspaces with restricted network egress. |

### API
| ID | Pri | Requirement |
|---|---|---|
| REQ-WEB-090 | SHOULD | Public REST API with API keys to create and query runs. |
| REQ-WEB-091 | SHOULD | Webhooks for run completion and escalation. |
| REQ-WEB-092 | MAY | GitHub Action to trigger a run from a PR or label. |

## 6. Non-functional requirements

| ID | Requirement |
|---|---|
| REQ-NFR-201 | p95 event latency runner to browser under 1 s; p95 mirror frame latency under 2 s (MVP). |
| REQ-NFR-202 | Availability target 99.5% for the control plane (MVP). Runner outages degrade gracefully with clear status. |
| REQ-NFR-203 | WCAG 2.2 AA for the web UI; keyboard-operable; respects `prefers-reduced-motion`. |
| REQ-NFR-204 | Evergreen browsers (latest two versions of Chrome, Edge, Firefox, Safari). Layout is usable at 1280 px and degrades gracefully on tablets; phone support is read-only for results. |
| REQ-NFR-205 | All user-facing text in English with an i18n-ready structure (Portuguese later). |

## 7. Success metrics (MVP)

- Activation: share of new orgs that complete a first run within 24 hours.
- Delivery rate: share of runs ending with a PR that a human merges (with or without edits).
- Time to first PR from "Run".
- Cost per run vs. price (gross margin).
- Escalation reasons distribution (guides roadmap).
- Weekly active projects.

## 8. Risks (product)

See `07-business-validation.md`. Highest: demand unvalidated; reliability across diverse customer repos; unit economics; trust to access source code; competition from funded products.
