# Global instructions

Load and follow `~/.copilot/AGENTS.md` for the persona fleet and orchestration rules. Repository-level instructions in `<repo>/.github/copilot-instructions.md` override these defaults.

## House rules (apply everywhere)

- Never merge a PR without me reading every review file in `reviews/`.
- Never touch secrets, env files, deploy config, or migrations without asking first.
- Reproduce every bug as a failing test first, then fix.
- Code and docs ship in the same PR. PR is not done otherwise.
- Prefer small, reversible changes. If the diff is over ~400 lines, propose a split before writing code.

## When to dispatch the fleet

When asked to review code, dispatch the persona fleet in parallel using `/fleet` and the model assignments in `~/.copilot/AGENTS.md`. Each persona writes its review to `reviews/<ticket>-<persona>.md` in the current repo. Sub-agents do not merge — I read all reports and decide.
