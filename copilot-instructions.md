# Global instructions

Load and follow `~/.copilot/AGENTS.md` for the persona fleet and orchestration rules. Repository-level instructions in `<repo>/.github/copilot-instructions.md` override these defaults.

## House rules (apply everywhere)

- Never merge a PR without me reading every review file in `reviews/`.
- Never touch secrets, env files, deploy config, or migrations without asking first.
- Reproduce every bug as a failing test first, then fix.
- Code and docs ship in the same PR. PR is not done otherwise.
- Prefer small, reversible changes. If the diff is over ~400 lines, propose a split before writing code.

## Persona invocation rule (hard — applies to every call)

Whenever any persona file under `~/.copilot/personas/` is referenced — by full path, by short name, via `/fleet`, via `/review`, or via a plain prompt — you MUST look up that persona's pinned model in the `## Model assignment` table inside `~/.copilot/AGENTS.md` BEFORE producing any review output.

Behaviour required:

1. **Fleet calls (`/fleet ...`):** dispatch each sub-agent on the model pinned for that persona. Different personas in the same fleet run on different models in parallel; never collapse them onto a single model "for convenience".
2. **Single-persona calls:** if the active session model already matches the pinned model, proceed. If it does not match, switch first with `/model <pinned-model>` and only then run the persona. Never silently run a persona on a model other than the one pinned in AGENTS.md.
3. **If `/model` is unavailable in the current execution context (e.g. inside a sub-agent),** stop and tell the user the exact command they need to run. Do not produce a degraded review.

The model table in `AGENTS.md` is the single source of truth. This rule is what makes that table actually binding.

## When to dispatch the fleet

When asked to review code, dispatch the persona fleet in parallel using `/fleet` and the model assignments in `~/.copilot/AGENTS.md`. Each persona writes its review to `reviews/<ticket>-<persona>.md` in the current repo. Sub-agents do not merge — I read all reports and decide.
