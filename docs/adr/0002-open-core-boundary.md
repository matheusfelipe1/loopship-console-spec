# ADR 0002: Open-core boundary

- **Status:** Accepted
- **Date:** 2026-10-29

## Context

The validator skill, implementer skill, CLI tools, and contracts are intended to be open source for credibility, adoption, and portfolio value. The hosted console is a potential product.

## Decision

**Open source (MIT or Apache 2.0, to be confirmed before publishing):** `mobile-qa` skill, `feature-implementer` skill, `qa-mobile-agent` CLI (including `loop decide`, `guard check`, `validate-contract`, `pr render`), JSON Schemas, templates, fixture app, and a **reference self-hosted runner**.

**Proprietary (hosted console):** web UI, multi-tenant control plane, billing, hosted runner fleet, team features (roles, audit log UI, share links), and operational tooling.

**Rule:** the console calls the open-source tools; it must not fork or reimplement verdict, loop, or guardrail logic. Behavior in the console and in local use stays identical.

## Consequences

- Users can start for free locally, trust the logic, and later pay for convenience, scale, and collaboration.
- Competitors can reuse the open parts; the moat is execution, reliability, UX, and trust.
- The self-hosted runner reduces security objections but lets customers avoid hosted runner fees; pricing should reflect that.
- License choice affects commercial copying risk; evaluate alternatives (for example AGPL for the console-adjacent parts) with legal advice before publishing.
