# copilot-fleet-starter

**Your pull request gets one AI review, from one model, on whatever you happened to have selected. This gives it eleven.**

Eleven specialists. Each pinned to the model that suits it. All running in parallel on the same diff, in their own context windows, none of them seeing what the others said. The bugs two of them flag without talking to each other are the ones that are actually broken.

It runs on GitHub Copilot CLI. You drop these files into `~/.copilot/` and your next session inherits the whole team.

## Why this exists

One AI reviewer is a generalist on a default model. It is polite. Polite is the problem. It comments on your variable names and waves the leaky endpoint through, because everyone has an opinion on naming and security is harder.

A real review board is not one reviewer. It is a security mind, an architect, a performance hawk, an engineer reading it cold, and someone who tries to break it. Different people. Different priorities. The disagreements are the point.

This repo is that board, standing and named, so you stop inventing roles one prompt at a time.

## What lands when you run it

You point it at a diff. A few minutes later, one file:

```
# Fleet summary — PR-142

## Triangulated (raised by 2+ personas — fix first)
- [TRIANGULATED] auth/login.js:88 — token compared after the DB call, not before
  (security-auditor, new-engineer)

## Conflicts (you decide)
- cache.js:40 — sceptical-architect wants the abstraction; performance-reviewer wants it inlined.

## Out of scope (in the diff, not in the ticket)
- README.md:12 — unrelated copy edit the ticket didn't ask for

## Everything else, ranked
### High
- api/users.js:51 — N+1 query on the members list (performance-reviewer)
```

Eleven reviews collapsed into one. Triangulated bugs at the top. Disagreements kept open, not buried. Scope creep called out. You read one page, not eleven.

## The team

Eleven personas. Each one has a narrow lane, a pinned model, and a list of things it is **not** allowed to comment on, so it stays in its lane. Call the whole fleet with `/fleet`, or call one by name when you only need that one set of eyes.

| Persona | Model | Lane | Call it when… |
|---|---|---|---|
| **security-auditor** | `gpt-5.3-codex` | Hostile input, auth order, secrets in logs, SSRF/SQLi/XSS | every diff that touches an endpoint, auth, or input handling |
| **prompt-injection-reviewer** | `gpt-5.3-codex` | Untrusted text reaching a prompt, a tool call, or an output | the code builds a prompt, calls an LLM, or acts on model output |
| **performance-reviewer** | `gpt-5.3-codex` | N+1 queries, allocations, wasted renders, hot loops | the diff touches a query, a list render, or a tight loop |
| **sceptical-architect** | `claude-opus-4.7` | Design pushback, the abstraction you'll regret, coupling | before you build something structural, or before a big merge |
| **data-engineer** | `claude-opus-4.7` | Schema, migrations, data contracts, backfills | the diff touches a migration, a schema, or a data shape |
| **ux-ui-researcher** | `claude-opus-4.7` | The screen *before* code: user, job-to-be-done, wireframe | you're about to build UI and haven't designed it yet |
| **ux-critic** | `claude-sonnet-4.6` | The built UI: what's confusing, in plain English | the diff changes something a user sees |
| **accessibility-reviewer** | `claude-sonnet-4.6` | WCAG, keyboard nav, screen-reader, contrast | any user-facing change, before it ships |
| **new-engineer** | `claude-haiku-4.5` | The cold read: unclear naming, missing context | always — it's cheap and catches what experts skim past |
| **qa-saboteur** | `gpt-5.4-mini` | Edge cases written as **real failing tests** and run | before any release, if a test runner is wired |
| **e2e-tester** | `claude-sonnet-4.6` | Launches the app, drives Playwright, iterates until green | before merge to main — the "did it work for a user?" gate |

### How to call each one

The whole fleet on a diff:

```
/fleet review the diff for PR-142 using the personas in ~/.copilot/personas/.
Use the model pinned in AGENTS.md. Write each review to reviews/PR-142-<persona>.md,
then write the Foreman summary to reviews/PR-142-SUMMARY.md.
```

One persona, by name (the orchestrator switches to its pinned model first):

```
# Security pass on a single file
review backend/auth/login.js with security-auditor

# Is this safe to feed an LLM?
review the prompt builder in ai/chat.js with prompt-injection-reviewer

# Design the screen before writing any code
use ux-ui-researcher to design the team-invite screen from this spec: <paste spec>

# Cold read for naming and clarity
review this diff with new-engineer

# Pushback before I commit to an abstraction
ask sceptical-architect whether this caching layer is worth it

# Turn the missed edge cases into failing tests and run them
use qa-saboteur on PR-142

# Drive the actual user journey end to end
use e2e-tester to verify the signup flow on the local dev server
```

Mix and match. A migration PR might be `security-auditor` + `data-engineer` + `new-engineer`. A new screen might be `ux-ui-researcher` first (pre-build), then `ux-critic` + `accessibility-reviewer` after. You decide the cast; the model pinning is automatic.

## Install (machine-wide, 2 minutes)

```bash
git clone https://github.com/sethiramicrosoft/copilot-fleet-starter.git
cp copilot-fleet-starter/copilot-instructions.md ~/.copilot/
cp copilot-fleet-starter/AGENTS.md ~/.copilot/
cp -r copilot-fleet-starter/personas ~/.copilot/
```

Windows PowerShell:
```powershell
Copy-Item .\copilot-instructions.md $env:USERPROFILE\.copilot\
Copy-Item .\AGENTS.md $env:USERPROFILE\.copilot\
Copy-Item .\personas $env:USERPROFILE\.copilot\ -Recurse
```

Open a new GHCP CLI session — it auto-loads the global instructions. Then point the fleet at your next PR.

### Optional: per-repo overrides

```bash
cp -r copilot-fleet-starter/repo-overlay/.github <your-repo>/
```
or run `/init` to generate one from scratch.

---

## What this repo gives you

```
~/.copilot/
├── copilot-instructions.md   ← global house rules, auto-loaded every session
├── AGENTS.md                  ← the fleet: cast + model-per-persona table + Foreman pass
└── personas/
    ├── security-auditor.md
    ├── prompt-injection-reviewer.md ← LLM trust boundaries (untrusted text → prompt/tool/output)
    ├── performance-reviewer.md
    ├── sceptical-architect.md
    ├── data-engineer.md
    ├── ux-ui-researcher.md    ← designs the screen BEFORE code (pre-build)
    ├── ux-critic.md           ← critiques the built UI (post-build)
    ├── accessibility-reviewer.md
    ├── new-engineer.md
    ├── qa-saboteur.md         ← writes & runs real failing tests when a runner is wired
    └── e2e-tester.md          ← drives Playwright, launches the app, iterates until green
```

Plus `repo-overlay/.github/copilot-instructions.md` — an example of repo-specific overrides you can drop into any project.

## What is in GHCP CLI vs what this adds

**Built into GHCP CLI (documented features):**
- `~/.copilot/copilot-instructions.md` auto-loaded for every session
- `<repo>/.github/copilot-instructions.md` per-repo overrides, generated by `/init`
- `AGENTS.md` recognised as an instruction file
- `/fleet` for parallel sub-agent dispatch
- `/review` with inline model pinning (e.g. `/review Use Opus 4.7 and Codex 5.3 ...`)
- `/model`, `/plan`, `/add-dir`, Autopilot, auto-compaction

**What this repo adds on top (discipline layer):**
- A standing cast of eleven named personas instead of inventing roles each time
- One of them (`prompt-injection-reviewer`) covers the LLM trust boundary that a plain security pass misses — for the increasing number of repos that call a model
- Two of those personas (`qa-saboteur`, `e2e-tester`) actually execute tests — Jest / Playwright — rather than only reviewing code
- A model-per-persona table so assignment is decided once
- A **Foreman pass** that reads all the reviews and writes one summary — triangulating findings raised by 2+ personas, logging conflicts instead of hiding them, and flagging scope creep
- A roster gate: the orchestrator prints which models it is about to run, before it spends them
- A convention that each persona appends lessons learned back into its own file
- A `reviews/<ticket>-<persona>.md` output convention so reviews survive the session

GHCP CLI gives you the **engine**. This repo is the **standing team** that runs on it.

### Going past static review — launch the app

`qa-saboteur` and `e2e-tester` are the two personas that don't stop at reading the diff:

- **qa-saboteur** — if a Jest / Vitest / Pytest runner is wired in the repo, it writes failing tests for the edge cases the diff missed and runs them. Output is real test files + a pass/fail summary, not prose.
- **e2e-tester** — if Playwright is configured, it launches the actual app (local dev server or a deployed environment), drives the user journey the ticket claims to deliver, captures screenshots / traces, and iterates until the journey is green or it has hard evidence of a bug.

Wire either runner once per repo and the personas pick it up automatically on the next `/fleet` run.

## What actually happens when you fire `/fleet`

Most people picture `/fleet` as "the session switches model N times in sequence." It isn't. Each persona runs in its own isolated context window on its own pinned model, in true parallel:

```
You ──► Orchestrator session (your /model, e.g. Sonnet 4.6)
            │
            │  prints the roster (persona → model) and waits for go
            │  reads AGENTS.md model table
            │  spawns N sub-agents in parallel
            │
            ├──► sub-agent: security-auditor    → gpt-5.3-codex        ──┐
            ├──► sub-agent: prompt-injection    → gpt-5.3-codex        ──┤
            ├──► sub-agent: performance-reviewer→ gpt-5.3-codex        ──┤
            ├──► sub-agent: sceptical-architect → claude-opus-4.7      ──┤
            ├──► sub-agent: data-engineer       → claude-opus-4.7      ──┤  all
            ├──► sub-agent: ux-ui-researcher    → claude-opus-4.7      ──┤ running
            ├──► sub-agent: ux-critic           → claude-sonnet-4.6    ──┤  at the
            ├──► sub-agent: accessibility       → claude-sonnet-4.6    ──┤  same
            ├──► sub-agent: new-engineer        → claude-haiku-4.5     ──┤  time
            ├──► sub-agent: qa-saboteur         → gpt-5.4-mini         ──┤
            └──► sub-agent: e2e-tester          → claude-sonnet-4.6    ──┘
                                                          │
                                                          ▼
                                              each writes its review to
                                              reviews/<ticket>-<persona>.md
                                                          │
            ┌─────────────────────────────────────────────┘
            ▼
       Orchestrator collects results, runs the Foreman pass,
       and shows you one reviews/<ticket>-SUMMARY.md
```

### The Foreman pass — one summary instead of eleven files

Eleven reviews in `reviews/` is eleven files to read. After the sub-agents finish, the orchestrator runs one more pass — the **Foreman** — that reads the reports (not the code) and writes a single `reviews/<ticket>-SUMMARY.md`:

- **Triangulation** — any finding raised independently by 2+ personas is promoted to the top and marked `[TRIANGULATED]`. Because the contexts are isolated, two personas landing on the same line without seeing each other is the strongest signal in the whole run. Fix those first.
- **Conflict log** — when the architect and the performance reviewer disagree, both positions are recorded side by side. The Foreman never silently picks a winner; the call stays yours. A buried disagreement is worse than an open one.
- **Scope check** — the Foreman compares the diff against the ticket and flags anything changed that the ticket did not ask for. No single persona watches for scope creep, because each one only sees its own lane.
- **One ranked list** — everything else, merged and severity-ranked, with the persona that raised it in brackets.

The Foreman runs on the orchestrator's own model — it is summarising, not reviewing.

### What this means in practice

| Property | Behaviour |
|---|---|
| **Model selection** | Each sub-agent process is launched with its pinned model from AGENTS.md. The orchestrator's own model is unaffected. |
| **Context isolation** | Each sub-agent has its own clean context window. The security auditor doesn't see what the UX critic is thinking, and vice versa. This is deliberate — it prevents groupthink. The triangulation effect (3 personas independently finding the same bug) only works because contexts are isolated. |
| **Parallelism** | True parallel, not round-robin. 11 personas don't take 11× one-persona time; they take ~max(individual times). Last live run: 5 personas → 2:22 wall clock vs. ~12 min if serial. |
| **Failure isolation** | If Opus is rate-limited that minute, only the architect/security/e2e sub-agents fail. The others complete. You can re-run just the failed ones. |
| **Cost** | Each sub-agent burns its own tokens at its own model's pricing. Opus calls cost Opus rates, Haiku calls cost Haiku rates. The cost math is what makes the model table valuable — you don't pay Opus for what Haiku can do. |
| **Token bloat in main session** | Zero — sub-agents don't dump their full work into your main context. Only a brief summary lands in your session. This is why you can run `/fleet` 5× in one session without auto-compaction firing. |

### What makes the model table *actually* binding

The model table in `AGENTS.md` is only honoured automatically because `copilot-instructions.md` includes a hard rule that says *"on every persona invocation — fleet OR single — look up the pinned model in AGENTS.md first."* Without that rule, single-persona calls silently fall through to whatever `/model` happens to be active. With it, you can just say *"review architecture using sceptical-architect"* and the orchestrator will switch to Opus 4.7 before responding. See `copilot-instructions.md` in this repo.

## References (GitHub docs)

- [CLI best practices](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices)
- [Fleet](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/fleet)
- [Autopilot](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/autopilot)

## License

MIT. Fork, copy, rewrite the personas to match your team.
