# ADR 0003: Runner isolation and credential handling

- **Status:** Accepted
- **Date:** 2026-10-29

## Context

Runners execute customer repositories, build scripts, and an AI agent that reads untrusted text. This is the highest-risk part of the system.

## Decision

- One job per ephemeral container or VM; no shared writable volumes; no privileged mode; resource and time limits.
- Outbound-only connectivity to the control plane; **egress allowlist** (package registries, the code host, tracker and design APIs, the model API, object storage). No access to cloud metadata endpoints or internal networks.
- Credentials are short-lived and scoped: GitHub installation token limited to the target repository; tracker and design tokens read-only unless write is enabled; model key injected as an ephemeral environment variable and never written to disk or logs.
- The agent can push a feature branch but cannot open or merge PRs: the control plane opens PRs after policy or approval.
- Workspaces are wiped on every exit path; artifacts are uploaded through pre-signed URLs scoped to the run.
- Self-hosted runners use the same agent, but the customer owns the network and hardware.

## Consequences

- Prompt injection or malicious build scripts have limited blast radius, but a determined attacker may still read the code of the same job; documentation states this honestly.
- Egress allowlists require maintenance per ecosystem (Gradle, Maven, CocoaPods, npm mirrors); provide per-project additions with review.
- Pushing a feature branch requires `contents:write`; recommend branch protection or rulesets on the base branch.
