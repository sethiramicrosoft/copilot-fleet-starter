# Persona fleet

Personas live in `~/.copilot/personas/*.md` machine-wide and in `.copilot/personas/*.md` per repo. Repo files override machine files for the same name.

## Cast

- `security-auditor` — paranoid, hostile-input-first
- `performance-reviewer` — N+1, allocations, wasted renders
- `sceptical-architect` — pushes back on design, names the regret
- `data-engineer` — schema, migrations, data contracts
- `ux-critic` — plain English, what is confusing
- `accessibility-reviewer` — WCAG, keyboard, screen reader
- `new-engineer` — reads cold, flags unclear naming
- `qa-saboteur` — static test design, edge cases as Jest/Vitest tests
- `e2e-tester` — launches the app, drives Playwright, iterates until green

## Model assignment

| Persona                | Model               | Why |
|------------------------|---------------------|-----|
| security-auditor       | `gpt-5.3-codex`     | Sharp on code patterns, cheap to run on every diff |
| performance-reviewer   | `gpt-5.3-codex`     | Catches N+1, wasted renders, allocation hotspots |
| sceptical-architect    | `claude-opus-4.7`   | Deepest reasoning, pushes back hardest on design |
| data-engineer          | `claude-opus-4.7`   | Schema and migration mistakes are expensive |
| ux-critic              | `claude-sonnet-4.6` | Fast, opinionated, plain English |
| accessibility-reviewer | `claude-sonnet-4.6` | Quick passes against WCAG patterns |
| new-engineer           | `claude-sonnet-4.6` | Reads code cold, flags unclear naming |
| qa-saboteur            | `gpt-5.4-mini`      | Cheap, generates ugly user behaviour fast |
| e2e-tester             | `claude-sonnet-4.6` | Strong tool-use + iteration loop; right balance for browser automation |

## When to dispatch the fleet

- **Before code:** send the spec to `sceptical-architect`, `ux-critic`, and one domain persona (e.g. `data-engineer` if it touches storage). Argue, refine, then build.
- **After code, before beta merge:** default fleet is `security-auditor`, `performance-reviewer`, `new-engineer`. Add `data-engineer` if the diff touches migrations or schemas. Add `ux-critic` and `accessibility-reviewer` if it touches anything user-facing. Add `qa-saboteur` before any release.
- **Before merge to main / production:** also run `e2e-tester` — it launches the app, drives Playwright through the affected user journeys, captures screenshots, and iterates until the suite is green. This is the "did it actually work for a user?" gate, not just "did the code review pass?".

## Orchestration

Dispatch as parallel sub-agents inside one CLI session using `/fleet`. Each sub-agent:

1. Reads its persona file from `~/.copilot/personas/<persona>.md` (or repo override).
2. Pins to the model in the table above.
3. Reads the diff (`git diff main...<branch>`).
4. Writes findings to `reviews/<ticket>-<persona>.md` with the persona's output format.
5. Appends any newly learned pattern to the **Lessons** section of its own persona file before exiting.

Sub-agents do not merge code. The human reads all reports and decides.

## Output format (enforced by every persona)

```
# <Persona> review of <ticket>

## Summary
<one paragraph>

## Findings
### <Severity: Critical | High | Medium | Low> — <one-line title>
- **Location:** `<file>:<line>`
- **Issue:** <what is wrong>
- **Patch:** <code block I can paste>

## Lessons (appended to persona file)
- <new pattern learned this review, if any>
```
