# Loopship Console: specification package

> Spec-Driven Development (SDD) package, v0.1. Language: English. **"Loopship" is a working title**: check trademarks and domain availability before using it publicly.

## What this is

The specification of a **web product**: paste a task link (Jira, Trello, Azure DevOps) and a design link (Figma), press **Run**, and watch an agent implement the feature, test it on a device, fix what fails, and open a pull request with evidence. A human reviews and decides.

It is a thin product layer over two open-source skills from the sibling packages:

| Package | Role |
|---|---|
| `qa-mobile-agent-spec` | `mobile-qa` validator skill + installer CLI + **contracts** |
| `feature-implementer-spec` | `feature-implementer` skill, loop decisions, guardrails |
| `loopship-console-spec` (this one) | Web console, orchestrator, hosted and self-hosted runners, live device mirror, billing |

**Open-core boundary:** skills, CLI, and contracts are open source (MIT/Apache). The console orchestrates them; it never reimplements their logic. See `docs/adr/0002-open-core-boundary.md`.

## Contents

```
specs/
  00-constitution.md
  01-product-requirements.md     # personas, stories, REQ-WEB-xxx
  02-architecture.md             # components, stack, data model, runners, mirror
  03-ux-and-design-system.md     # dark, developer-tool UI; tokens; screens
  04-api-and-events.md           # REST, event stream, runner protocol
  05-tasks.md                    # milestones and tasks (work queue)
  06-testing-ops-security.md
  07-business-validation.md      # pricing hypotheses, validation plan, risks
contracts/                       # copies of the shared JSON Schemas (read-only here)
prototype/                       # interactive dark UI prototype (open index.html)
docs/adr/                        # decision records
CLAUDE.md                        # rules for the build agent
```

## Prototype

Open `prototype/index.html` in a browser. It is a self-contained, scripted mock (no backend): it shows the New Run form, a live run with the pipeline, log stream, **device mirror**, findings, verdict, and the cycle where a NO-GO becomes GO. See `prototype/README.md`.

## How to build this with Claude Code (SDD workflow)

1. Create a new repository and copy this package into it.
2. Prompt: *"Read `CLAUDE.md`, then `specs/` in numeric order. Implement the next unchecked task in `specs/05-tasks.md`. Stop after its Definition of Done."*
3. Review each task, commit, and continue.
4. Before building the hosted parts, complete the **validation tasks** in `specs/07-business-validation.md` (interviews, concierge pilot). Building the full product before talking to users is the main risk of this plan.

## Recommended order across the three packages

1. Build `qa-mobile-agent-spec` (validator + CLI).
2. Build `feature-implementer-spec`.
3. Publish both as open source and write the article.
4. Validate demand (`07-business-validation.md`).
5. Only then build this console, starting with the MVP slice defined in `05-tasks.md` (Android, Jira + Figma + GitHub, one hosted runner).

## Out of scope for v1

iOS hosted runners (v1.2), GitLab/Bitbucket, SSO/SAML, on-prem installation, Adobe XD (in maintenance mode), automatic merge or deploy.
# loopship-console-spec
