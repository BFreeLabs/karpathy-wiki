# Prompt: Upgrade From Template

> Use this prompt by saying *"run the upgrade-from-template prompt"*, *"check for methodology updates"*, or *"sync from upstream template"*.

This prompt pulls methodology improvements from the upstream template repo (`BFreeLabs/karpathy-wiki`) into the current wiki **without touching any topic-specific content**. The user runs it whenever they want to know what's new upstream and pick which improvements to adopt.

The whole point: methodology evolves over time. New rules, better prompts, sharper templates. You should not have to manually diff folders or remember which files were updated when. The agent does the comparison and presents a curated list.

## Boundary — what this prompt may and may not touch

**May modify:**
- `.instructions/core/` (everything under it)
- `wiki/log.md` (a single new entry recording the sync)
- `CLAUDE.md` **only** when the upstream change requires a corresponding row in the prompts/templates/rules tables — and only those table rows, never the Wiki Scope or project declarations

**Must never modify:**
- `wiki/` (other than appending to `log.md`)
- `raw/`
- `.instructions/projects/`
- `CLAUDE.md` Wiki Scope, Directory Structure project markers, or any project-specific declaration
- `README.md`
- `quartz.config.ts`
- `.github/` (workflows, scripts, issue/PR templates — those are generic but versioned per-installation)

If a proposed upstream change would touch any "must never" path, **flag it and stop** — the user makes the call manually, never the agent.

## Phase 1 — Set up the upstream remote (one time)

Before the first run, the upstream template repo must be reachable as a git remote named `template`. Check:

```bash
git remote get-url template 2>/dev/null
```

If missing, propose adding it and wait for user approval:

```bash
git remote add template https://github.com/BFreeLabs/karpathy-wiki.git
git fetch template
```

If the user is using their own fork of the template, ask them which URL to use before running the `git remote add`.

## Phase 2 — Fetch and diff

```bash
git fetch template
git diff --name-status template/main...HEAD -- .instructions/core/
git diff --name-status HEAD...template/main -- .instructions/core/
```

The second diff (HEAD...template/main) shows what's new upstream. For each file listed:

- **Added (`A`)** — a new file exists upstream that the local wiki doesn't have
- **Modified (`M`)** — the file exists in both but has changed upstream
- **Deleted (`D`)** — the file was removed upstream (rare; verify before applying)

For each modified file, show the user a one-paragraph summary of what changed. Use:

```bash
git log --oneline HEAD..template/main -- .instructions/core/<file>
git diff HEAD...template/main -- .instructions/core/<file>
```

If a file has many small changes, summarize the *intent* (e.g., "tightened the dangling-link filter to skip inline code spans") rather than dumping the raw diff.

## Phase 3 — Present and pause (REQUIRED)

Produce a single structured report:

```markdown
## Upstream methodology updates

### New files (N)
- `.instructions/core/rules/foo-rules.md` — one-line description from upstream commit
- `.instructions/core/prompts/bar.md` — one-line description

### Modified files (N)
- `.instructions/core/prompts/lint.md` — added size-compliance check (Phase 1 step 11)
- `.instructions/core/rules/page-conventions.md` — clarified org-page condition (b)

### Deleted upstream (N)
- `.instructions/core/templates/old-thing.md` — superseded by new-thing.md

### CLAUDE.md table updates required
- Add row to Rules table for `foo-rules.md`
- Add row to Reusable prompts table for `bar.md`

### Out-of-scope changes detected (will NOT apply)
- `quartz.config.ts` — base styling change. Apply manually if you want.
- `README.md` — wording tweaks. Apply manually if you want.
```

**Stop here. Wait for the user to approve which items to apply.** They may approve all, approve a subset, defer some, or reject some entirely.

## Phase 4 — Apply approved changes

For each approved file:

```bash
git checkout template/main -- .instructions/core/<file>
```

For approved CLAUDE.md table updates, edit only the affected table rows. Never touch the Wiki Scope or Directory Structure.

## Phase 5 — Verify and log

1. Run `git status` and report which files changed.
2. Append a single entry to `wiki/log.md`:
   ```markdown
   ## [YYYY-MM-DD] upgrade-from-template
   - Synced from `template/main` at <commit-sha>
   - Pulled: <list of files>
   - Updated CLAUDE.md tables for: <list of additions>
   - Deferred: <list of items the user said no to>
   ```
3. Propose a single commit:
   ```
   chore: pull upstream .instructions/core/ improvements

   - <list of files>

   Source: BFreeLabs/karpathy-wiki@<commit-sha>
   ```
4. Wait for the user to approve the commit. Do not push.

## Don'ts

- **Never touch `wiki/` content.** The whole point of the boundary is that your wiki content stays yours.
- **Never apply "out-of-scope" changes silently.** Surface them in the report and let the user decide manually.
- **Never auto-resolve merge conflicts.** If `git checkout template/main -- <file>` would clobber local edits the user made to that core file, stop and ask. Local edits to `.instructions/core/` are unusual but legal — the user may have customized a prompt — and you must not lose them.
- **Never run this prompt against a wiki that isn't a template clone.** If `git remote get-url template` fails *and* the user can't tell you the upstream URL, stop. This prompt is only for downstream forks of the template.
- **Never push.** The user pushes when they're ready.

## When to suggest running this prompt

Mention this prompt to the user proactively if:
- It's been a long time since the last `upgrade-from-template` log entry (months)
- A lint pass surfaces a problem that the upstream methodology has already fixed (you noticed because the upstream commit message names the bug)
- The user mentions they saw a new feature in the template repo's release notes

## See Also

- [`customize-template.md`](customize-template.md) — the one-time onboarding prompt this complements
- The Rules and Templates tables in `CLAUDE.md` — the registries that may need new rows after a sync
