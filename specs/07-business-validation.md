# 07. Business validation

> Can this make money? **Possibly, but it is not guaranteed.** The market has funded competitors and the technical risks (reliability across diverse repositories, cost per run) are real. This document turns the uncertainty into cheap experiments you run **before** building the hosted product. Numbers below are hypotheses to test, not facts.

## 1. Competitive reality (researched October 2026)

See `qa-mobile-agent-spec/docs/market-landscape.md`. In short: commercial agentic mobile QA platforms exist (for example Autosana, backed by Y Combinator, with a US$3.2M round announced in February 2026), device automation is being commoditized by open-source MCP servers and Maestro, and coding agents are adding their own workflows. Differentiation must come from the **complete loop** (tracker + design as sources of truth, independent validation, deterministic gate, evidence, PR) and from **trust** (open core, self-hosted runners, honest limits).

## 2. Customer hypotheses

| Segment | Why they might pay | Why they might not |
|---|---|---|
| Small and mid mobile teams (5 to 50 engineers) without QA automation | Save review and manual testing time; evidence for PRs | Tasks too varied; trust in code access |
| Agencies and consultancies | Faster delivery, proof of testing for clients | Margin pressure; custom stacks |
| QA teams | Automate repetitive validation and regression authoring | Prefer deterministic suites and existing tools |
| Design-led product teams | Catch design drift early | Visual comparison not pixel-exact |

Primary hypothesis: **mobile teams will pay to offload small, well-specified tasks if delivery rate and trust are high enough.** Secondary: **QA validation alone (without implementation) is an easier first product.**

## 3. Offer and pricing hypotheses (to test)

| Option | Idea | Notes |
|---|---|---|
| Free / open source | CLI + skills, self-run | Adoption and credibility; primary growth channel |
| Team plan | Seats + included runner minutes + usage overage | Predictable for buyers |
| Usage-based | Price per completed run or per runner minute | Aligns with your cost; harder to budget for customers |
| Bring your own model key | Customer pays model costs directly | Lowers your cost and risk; simplifies launch |
| Self-hosted runner | Lower price, customer provides hardware | Addresses security; reduces your infrastructure cost |

Do not publish prices until you know your cost per run. Cost per run = model usage + runner minutes + storage and egress + support allocation. Measure it in the concierge pilot and in T-132 of the implementer package.

Target (hypothesis): gross margin high enough to sustain support and infrastructure (decide the threshold after measuring; many software businesses aim for 70% or more, but agent workloads can start lower).

## 4. Validation plan (maps to tasks T-001 to T-003)

### Interviews (10 to 15)
Script highlights: how tasks flow today; how features are validated on devices; what breaks most often; tools in use; what they tried with coding agents; what would make them trust an agent with a branch; what they would pay and who approves the budget; security blockers.

### Landing page and waitlist
Message: "Ticket + design in, tested pull request out, with evidence." Show the prototype video and the honest limits. Track sign-ups with a privacy-respecting counter.

### Concierge pilot (3 teams)
Run the open-source skills for them on real tasks. Measure delivery rate (PRs merged), edits needed, time saved (self-reported), failures, and cost per run. Ask for a written commitment (letter of intent or paid pilot) at the end.

### Decision thresholds for gate G1 (adjust to your context)
- At least 3 teams express willingness to pay or sign a paid pilot/LOI.
- Delivery rate on pilot tasks is high enough that reviewers say it saves time (collect their rating).
- Measured cost per run leaves room for a viable price.
- No unresolvable security objection among the pilots.

If thresholds fail, options: narrow to QA-only validation, pivot to a consulting/services offer around the open-source tools, or stop and keep the open-source projects as portfolio.

## 5. Go-to-market (if validated)

1. Open-source skills and the article (credibility, inbound).
2. Community presence: mobile developer groups, QA communities, conference talks.
3. Content: case studies from pilots (with permission), cost and reliability numbers, "what failed" posts.
4. Partnerships: tracker/design/CI ecosystems, consultancies.
5. Sales motion: product-led for small teams, founder-led for larger ones.

## 6. Key risks and mitigations

| Risk | Mitigation |
|---|---|
| Demand unproven | Gate G1 before building |
| Reliability varies across repos | Start with a narrow slice (Android + Jira + Figma + GitHub), curated onboarding, clear "good task" guidelines |
| Cost per run too high | Budgets, bring-your-own-key, smaller scope tasks, measure early |
| Trust and security objections | Open core, self-hosted runners, minimal permissions, transparency, security page |
| Platform dependence (model provider, MCP servers, tracker APIs) | Replaceable agent engine, adapter boundaries, pinned versions |
| Competition | Focus on the full loop, openness, and a specific niche first |
| Solo-founder capacity | Cut scope aggressively; consider managed services; define stop conditions |

## 7. Personal/career framing

Even if the business does not materialize, the open-source packages, the article, and the prototype are a strong portfolio. Set a time box for each phase and decide in advance what result would make you continue.

## 8. Legal and employment checks

- Review your employment contract for exclusivity or IP clauses before starting a commercial product.
- Do not use employer code, tickets, or designs in demos, tests, or content.
- Trademark and domain check for the final product name.
- Consult a professional for terms, privacy, and data processing agreements before taking customers.
