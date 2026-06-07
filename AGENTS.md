# Persona fleet

Personas live in `~/.copilot/personas/*.md` machine-wide and in `.copilot/personas/*.md` per repo. Repo files override machine files for the same name.

## Cast

- `security-auditor` — paranoid, hostile-input-first
- `prompt-injection-reviewer` — untrusted text → prompt/tool/output trust boundaries (for any code that calls an LLM)
- `performance-reviewer` — N+1, allocations, wasted renders
- `sceptical-architect` — pushes back on design, names the regret
- `data-engineer` — schema, migrations, data contracts
- `ux-ui-researcher` — designs the screen before code (user, job-to-be-done, wireframe)
- `ux-critic` — plain English, what is confusing
- `accessibility-reviewer` — WCAG, keyboard, screen reader
- `new-engineer` — reads cold, flags unclear naming
- `qa-saboteur` — static test design, edge cases as Jest/Vitest tests
- `e2e-tester` — launches the app, drives Playwright, iterates until green

## Model assignment

| Persona                | Model               | Why |
|------------------------|---------------------|-----|
| security-auditor       | `gpt-5.3-codex`     | Sharp on code patterns, cheap to run on every diff |
| prompt-injection-reviewer | `gpt-5.3-codex`  | Same code-pattern sharpness, aimed at LLM trust boundaries |
| performance-reviewer   | `gpt-5.3-codex`     | Catches N+1, wasted renders, allocation hotspots |
| sceptical-architect    | `claude-opus-4.7`   | Deepest reasoning, pushes back hardest on design |
| data-engineer          | `claude-opus-4.7`   | Schema and migration mistakes are expensive |
| ux-ui-researcher       | `claude-opus-4.7`   | Design decisions are expensive to undo; needs deepest reasoning |
| ux-critic              | `claude-sonnet-4.6` | Fast, opinionated, plain English |
| accessibility-reviewer | `claude-sonnet-4.6` | Quick passes against WCAG patterns |
| new-engineer           | `claude-haiku-4.5`  | Reads code cold, flags unclear naming — cheap pass, latest haiku |
| qa-saboteur            | `gpt-5.4-mini`      | Cheap, generates ugly user behaviour fast |
| e2e-tester             | `claude-sonnet-4.6` | Strong tool-use + iteration loop; right balance for browser automation |

## When to dispatch the fleet

- **Before code:** send the spec to `ux-ui-researcher`, `sceptical-architect`, and one domain persona (e.g. `data-engineer` if it touches storage). Argue, refine, then build. `ux-critic` is post-build and does not run here.
- **After code, before beta merge:** default fleet is `security-auditor`, `performance-reviewer`, `new-engineer`. Add `data-engineer` if the diff touches migrations or schemas. Add `prompt-injection-reviewer` if the diff touches any code that builds a prompt, calls an LLM, or acts on model output. Add `ux-critic` and `accessibility-reviewer` if it touches anything user-facing. Add `qa-saboteur` before any release.
- **Before merge to main / production:** also run `e2e-tester` — it launches the app, drives Playwright through the affected user journeys, captures screenshots, and iterates until the suite is green. This is the "did it actually work for a user?" gate, not just "did the code review pass?".

## Orchestration

Dispatch as parallel sub-agents inside one CLI session using `/fleet`. Before dispatch, the orchestrator prints the **roster** — the persona → model list it is about to launch — and waits for the go-ahead. This is the only cost gate the fleet has: you see which models are about to run, on which personas, before any tokens are spent. No hidden fleet.

Each sub-agent:

1. Reads its persona file from `~/.copilot/personas/<persona>.md` (or repo override).
2. Pins to the model in the table above.
3. Reads the diff (`git diff main...<branch>`).
4. Writes findings to `reviews/<ticket>-<persona>.md` with the persona's output format.
5. Appends any newly learned pattern to the **Lessons** section of its own persona file before exiting.

Sub-agents do not merge code. The human reads all reports and decides.

## The Foreman pass

After the sub-agents finish, the orchestrator runs one final pass — the Foreman — and writes a single `reviews/<ticket>-SUMMARY.md`. The Foreman does not review the code. It reads the other reports and does four things:

1. **Triangulation.** Any finding raised independently by two or more personas is promoted to the top of the summary and marked `[TRIANGULATED]`. Isolated contexts make this signal real — when the security auditor and the new engineer both flag the same line without seeing each other, that line is almost certainly broken. Fix these first.
2. **Conflict log.** When two personas disagree (the architect wants the abstraction, the performance reviewer wants it inlined), the Foreman records both positions side by side under `## Conflicts` and leaves the call to the human. It never silently picks a winner. A buried disagreement is worse than an open one.
3. **Scope check.** The Foreman compares the diff against the ticket's stated scope and flags any change that is outside it under `## Out of scope`. Scope creep is the failure no single persona is looking for, because each one only sees its own lane. Folding the check in here costs nothing and needs no extra persona.
4. **One ranked list.** Everything else is merged into a single severity-ranked list with the persona that raised it in brackets, so the human reads one file instead of eleven.

The Foreman runs on the orchestrator's own model — it is summarising, not reviewing, so it does not need a pinned heavy model.

### Foreman output format

```
# Fleet summary — <ticket>

## Triangulated (raised by 2+ personas — fix first)
- [TRIANGULATED] `<file>:<line>` — <issue> (security-auditor, new-engineer)

## Conflicts (human decides)
- `<file>:<line>` — sceptical-architect wants <X>; performance-reviewer wants <Y>.

## Out of scope (in the diff, not in the ticket)
- `<file>:<line>` — <what was changed that the ticket did not ask for>

## Everything else, ranked
### Critical
- `<file>:<line>` — <issue> (persona)
### High
- ...
```

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
