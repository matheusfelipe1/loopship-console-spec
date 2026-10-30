# Prototype

`index.html` is a **self-contained interactive mock** of the Loopship Console. It has no backend and no build step: open it in a modern browser.

## What it demonstrates

- Dark, developer-tool visual language (tokens from `specs/03-ux-and-design-system.md`).
- **New run** form with task/design link detection (Jira, Trello, Azure DevOps, Figma; Adobe XD flagged as unsupported).
- **Live run** view: pipeline stepper, event timeline and log stream, **device mirror** with tap markers, cycle counter, budget meters.
- A scripted scenario: cycle 1 finds a crash and a design divergence (NO-GO), cycle 2 fixes the crash (GO with caveats), then the approval modal and "PR opened".
- **Result** tabs: findings, design comparison (Figma vs real with a slider), evidence, report.
- Command palette (`Ctrl/Cmd + K`), keyboard shortcuts, reduced-motion support.

## Controls

- `Run` (New run form or the top bar) starts the scripted scenario.
- `Speed` toggles 1x, 2x, 4x playback.
- `Reset` returns to the initial state.
- `Ctrl/Cmd + K` opens the command palette; `N` focuses New run; `Esc` closes overlays.

## Notes for the build

- Mirror frames in the prototype are drawn with HTML/CSS; in the product they arrive over the mirror channel (`specs/04-api-and-events.md` section 4).
- Event names in the log mirror the event types in `specs/04-api-and-events.md`.
- The data shown is fictional.
