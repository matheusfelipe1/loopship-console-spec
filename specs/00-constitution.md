# 00. Constitution (Loopship Console)

Inherits the constitutions of `qa-mobile-agent-spec` and `feature-implementer-spec`. Additional principles for a hosted product:

1. **Trust is the product.** Customers hand over source code and design access. Least privilege, short-lived credentials, clear data retention, and honest security claims come before features.
2. **The console is a thin layer.** It orchestrates open-source skills and tools. It does not fork their logic, so the open-source and hosted behaviors stay identical.
3. **Humans decide.** Every run ends in a pull request or an escalation. The console offers "Approve and open PR", "Request changes", and "Escalate to a person". It never merges or deploys.
4. **Show the work.** Live timeline, device mirror, evidence, and an audit trail. Nothing important happens off-screen.
5. **Bounded cost.** Every run has budgets (cycles, minutes, spend). Customers can cap spending per run and per month. No surprise bills.
6. **Isolation by default.** Each job runs in an ephemeral, isolated workspace with restricted network egress. Tenants never share workspaces, caches with source code, or runners without a clean wipe.
7. **Untrusted content stays untrusted.** Ticket text, design text, app UI, and agent output are rendered escaped and never treated as instructions.
8. **Own your exit.** Data export and a self-hosted runner option exist from the start. Contracts are open.
9. **Honest positioning.** The console says what it cannot do (speed, non-determinism, model-based visual comparison, iOS limits) rather than overselling.
10. **Validate before scaling.** Build the smallest slice that can prove demand, then expand.

## Non-goals (v1)

- Replacing CI/CD, test clouds, or issue trackers.
- Automatic merge or deployment.
- On-prem installation, SSO/SAML, and data residency choices (planned later).
- Supporting every tracker or design tool. v1 targets Jira, Figma, GitHub.
- Real-time video streaming at 60 fps. The mirror is "good enough to follow the run" (see architecture).

## Product quality bar

- First run from sign-up in under 10 minutes using the managed Android runner and a sample project.
- A reviewer can understand any run from the UI alone, without reading logs.
- p95 event latency from runner to browser under 1 second; mirror frame latency under 2 seconds in the MVP.
- No cross-tenant data access, verified by automated tests.

## Glossary additions

| Term | Meaning |
|---|---|
| Organization (Org) | Billing and access boundary |
| Project | A repository plus configuration (commands, limits, targets) |
| Run | One execution from task link to PR or escalation |
| Runner | A worker that executes jobs: hosted by us or self-hosted by the customer |
| Job | A unit of work leased to a runner |
| Mirror | Near-real-time view of the device screen in the browser |
| Approval | A human decision requested by the run |
