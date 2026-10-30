# 03. UX and design system

The product is a **developer tool**. The interface should feel precise, fast, and calm: dark by default, dense but readable, keyboard-first, with real-time feedback. The interactive reference is `prototype/index.html`. Match its tokens and layout.

## 1. Principles

1. **Show the work.** Timeline, mirror, evidence, and verdict are first-class. Never hide progress behind a spinner.
2. **Status at a glance.** One consistent status vocabulary and color semantics across lists, headers, and chips.
3. **Keyboard-first.** Command palette, shortcuts, visible focus.
4. **Progressive disclosure.** Simple run form first; limits and advanced options on demand.
5. **Truthful UI.** Say when something is a model judgment (design comparison) or an estimate (cost).
6. **Calm motion.** Short transitions that explain change; honor reduced motion.

## 2. Tokens (dark theme)

### Color

| Token | Value | Use |
|---|---|---|
| `--bg` | `#0A0C10` | App background |
| `--surface-1` | `#10131A` | Panels, sidebar |
| `--surface-2` | `#161A23` | Cards, inputs |
| `--surface-3` | `#1D2230` | Hover, selected rows |
| `--border` | `#262C3B` | Dividers, outlines |
| `--border-strong` | `#343C50` | Focused/hover outlines |
| `--text` | `#E7EAF3` | Primary text |
| `--text-2` | `#A3ACC2` | Secondary text |
| `--text-3` | `#6F7891` | Muted/disabled |
| `--accent` | `#7C6CFF` | Primary actions, active states |
| `--accent-2` | `#34D5C4` | Highlights, live indicators |
| `--success` | `#3DD68C` | GO, passed |
| `--warning` | `#F6B84A` | GO with caveats, warnings |
| `--danger` | `#FF5F6D` | NO-GO, failed, blocking |
| `--info` | `#5EA9FF` | Informational |

Use semantic colors only for status, never decoration. Soft backgrounds are the semantic color at 12-16% alpha over `--surface-2`. Minimum text contrast 4.5:1 (3:1 for large text and UI boundaries).

### Typography

- UI: **Inter** (fallback: system UI). Mono: **JetBrains Mono** (fallback: ui-monospace) for logs, IDs, code.
- Scale (px / line-height): 12/16 caption, 13/20 UI small, 14/22 body, 16/24 emphasis, 20/28 section, 28/34 page title.
- Weights: 400 body, 500 labels, 600 headings. Tabular numerals for metrics.

### Spacing, radius, elevation, motion

- Spacing scale: 4, 8, 12, 16, 24, 32, 48. Panel padding 16; page gutters 24.
- Radius: 6 (chips, inputs), 10 (cards), 14 (device frame, modals).
- Elevation: borders over shadows; modals use a 1px border plus a soft shadow (`0 12px 40px rgba(0,0,0,.45)`).
- Motion: 120 ms (hover), 200 ms (panels), ease-out; skeleton shimmer 1.4 s; mirror frames cross-fade 120 ms. Disable non-essential motion under `prefers-reduced-motion`.
- Focus ring: 2px `--accent` with 2px offset on all interactive elements.

## 3. Layout

```
+-----------+----------------------------------------------------------+
| Sidebar   | Top bar: breadcrumb | project switcher | search (Cmd+K) |  |
| Runs      +----------------------------------------------------------+
| Projects  |                                                          |
| Runners   |                    Page content                          |
| Integr.   |                                                          |
| Usage     |                                                          |
| Settings  |                                                          |
+-----------+----------------------------------------------------------+
```

Sidebar 232 px (collapsible to 56 px). Content max width 1440 px. Run detail uses a three-column layout: timeline/log (left, 360 px), device mirror (center, flexible), findings and evidence tabs (right, 400 px). Below 1100 px the right column becomes tabs under the mirror.

## 4. Components

Button (primary, secondary, ghost, danger), Input, URL input with provider chip, Select, Segmented control, Switch, Chip/Badge (status), Tabs, Table with sticky header, Toast, Modal, Tooltip, Command palette, Stepper (pipeline), Timeline item, Log viewer (virtualized, follow mode), Device frame (phone with notch, rounded 14 px) with tap-marker overlay, Image comparison (side by side + slider), Video player, Verdict banner, Metric tile, Empty state, Skeleton.

### Status chips

| Status | Color | Label |
|---|---|---|
| queued | `--text-3` | Queued |
| running | `--accent-2` (pulsing dot) | Running + step |
| succeeded / GO | `--success` | Succeeded / GO |
| GO with caveats | `--warning` | Caveats |
| NO-GO / failed | `--danger` | NO-GO / Failed |
| escalated | `--warning` | Needs you |
| cancelled | `--text-3` | Cancelled |

## 5. Screens

1. **Runs (home):** table (status, issue key and title, project, platform, cycle, verdict, duration, cost, started). Filters, search, "New run" primary button. Empty state teaches the first run.
2. **New run:** task URL and design URL inputs with live provider detection chips (Jira, Trello, Azure DevOps, Figma; unsupported tools show a clear note). Project, platform (Android/iOS), target, mode segmented control (Implement and validate / Validate only), collapsible "Limits" (max cycles, minutes, spend), Run button with cost/time estimate disclaimer.
3. **Run detail (live):** header (issue, status chip, cycle `2/3`, elapsed, cost, Cancel), pipeline stepper (Read, Plan, Implement, Verify, Build, Validate, Decide, Deliver), left timeline/log, center device mirror with tap markers and step caption, right tabs (Findings, Design, Evidence, Report).
4. **Result view:** same page after completion; verdict banner (GO / Caveats / NO-GO / Escalated) with reasons; criteria table; bugs with severity and evidence; regression table; design comparison with slider; cycle history; actions: "Approve and open PR", "Request changes", "Open PR", "Export report".
5. **Approval modal:** shows the question, context, options; decision recorded in the audit log.
6. **Projects:** list and detail (repo, base branch, commands, limits, targets, validated config status).
7. **Integrations:** cards with connection state, scopes granted, and "what this can access" disclosure; GitHub App install; Jira/Figma OAuth.
8. **Runners:** hosted pool status, self-hosted runner list, "Add runner" with one-time command; health and version.
9. **Usage and billing:** consumption by project and run; spend cap controls.
10. **Settings:** team and roles, API keys, retention, audit log.

## 6. States

Every data view defines: loading (skeleton), empty (explain and offer the next action), error (what failed, retry), and permission-denied (who can grant access). Live views define "disconnected" (auto-reconnect with backoff, banner) and "stale" (runner heartbeat lost).

## 7. Accessibility

- WCAG 2.2 AA; semantic landmarks; focus order follows visual order; focus trapped in modals.
- Status never conveyed by color alone (icon + text).
- Mirror has an accessible text description of the current step; timeline is an ARIA live region (polite).
- Keyboard: `Cmd/Ctrl+K` palette, `G R` go to runs, `N` new run, `/` search, `[` toggle sidebar, `Esc` close.

## 8. Content tone

Short, specific, and calm. Prefer verbs and facts ("Build failed: Gradle task :app:assembleDebug") over mood. Never say "AI magic". Disclose limits ("Design comparison is a visual judgment, not a pixel diff").

## 9. Brand (placeholder)

Working title "Loopship". Mark: a small looping arrow around a ship-like arrow/triangle in `--accent`. Replace after naming and trademark checks.
