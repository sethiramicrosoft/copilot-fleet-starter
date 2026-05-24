# Repo instructions (example overlay)

Inherit everything from `~/.copilot/copilot-instructions.md` and `~/.copilot/AGENTS.md`. The overrides below apply only inside this repository.

## Stack

- React frontend in `/frontend`
- Node backend in `/backend`
- Postgres
- Scheduled jobs in `/jobs`

## Deploy rules

- Merge to `main` triggers a beta deploy.
- Prod is manual cherry-pick only.
- Never deploy on a Friday after lunch.

## Repo-specific persona overrides

- `security-auditor` runs in **paranoid mode** here: flag *any* new endpoint without rate limiting, even internal ones.
- `data-engineer` runs against the Postgres migrations folder explicitly; assume the table sizes in `docs/data-sizes.md`.
- `accessibility-reviewer` enforces WCAG **AA** as a release blocker, not a nice-to-have.

## Reviews folder

All fleet reviews land in `reviews/<ticket>-<persona>.md` at the repo root. Reviews are committed alongside the code change in the same PR.
