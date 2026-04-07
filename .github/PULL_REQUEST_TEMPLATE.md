# Pull Request

Thanks for contributing to the Karpathy Wiki template. Pick the path that matches your change.

## Methodology change (most common)

PRs that touch only `.instructions/core/` are the heart of this project — they benefit every downstream wiki.

- **Files touched**: list the paths under `.instructions/core/`
- **What changed**: one paragraph
- **Why**: the concrete situation that prompted it (link to an issue if there is one)
- **Migration impact for downstream wikis**: zero-touch / requires CLAUDE.md table update / requires manual step (specify)

## Tooling change

PRs that touch `.github/workflows/`, `.github/scripts/`, `.github/ISSUE_TEMPLATE/`, or `quartz.config.ts`.

- **What changed**: one paragraph
- **Tested how**: local build, dispatched workflow, etc.

## Documentation change

PRs that touch only `README.md` or `.github/PULL_REQUEST_TEMPLATE.md`.

- **What changed**: one sentence

## Out-of-scope (will be rejected)

- Changes to `wiki/` — that's downstream content, not template content
- Changes to `CLAUDE.md` Wiki Scope or project declarations — those are per-installation
- Topic-specific rules, prompts, or templates — put them in your own fork's `.instructions/projects/`
