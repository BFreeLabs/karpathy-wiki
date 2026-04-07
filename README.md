# Karpathy Wiki

**A GitHub template for an LLM-maintained second brain.** Inspired by [Andrej Karpathy's LLM Wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). You curate the sources; the LLM does the writing, cross-referencing, linting, and filing. The whole wiki publishes as a static site to GitHub Pages.

This template is **topic-agnostic**. The methodology in `.instructions/core/` works for any subject — AI tools, food science, medieval history, home-lab infrastructure, paper reading lists, anything. The customize-template prompt walks you through scoping it to your topic in about 10 minutes.

The best part: you don't have to memorize any commands to set this up. Just tell your agent what you want, and it does the rest. In a fresh Claude Code (or any agent that can run `gh` and `git`) session, paste:

> *"Install the BFreeLabs/karpathy-wiki template into ~/Documents/my-second-brain, then run the customize-template prompt."*

The agent will fork the template into your account, clone it locally, read [`.instructions/core/prompts/customize-template.md`](.instructions/core/prompts/customize-template.md), and walk you through scoping the wiki to your topic. From clone to first ingest is one paragraph of natural language.

---

## What's in this template

```
your-wiki/
  CLAUDE.md          ← topic scope + rules (the only file you customize by hand)
  README.md          ← this file
  .instructions/
    core/            ← data-agnostic methodology — DO NOT manually edit; sync from upstream
      prompts/       ← callable prompts (customize-template, ingest, lint, upgrade-from-template, …)
      rules/         ← page conventions, size budgets, PDF extraction
      templates/     ← page templates (source-summary, youtube-source, pdf-source, person-page)
    projects/        ← your project-specific methodology (created by customize-template)
    tools/           ← gitignored, per-installation helpers
  raw/               ← source documents waiting for ingest (gitignored)
    archive/         ← already-ingested sources
    assets/          ← attachments
  wiki/              ← LLM-generated and LLM-maintained markdown (the published site)
    {hot,index,log,overview,tasks}.md
    people/ orgs/ tools/ open-source/ concepts/ analyses/ sources/
  quartz.config.ts   ← Quartz v4 build config (set pageTitle + baseUrl)
  .github/           ← GitHub Actions workflow for Pages publishing
```

**The key separation**: `.instructions/core/` is generic methodology — never edit it by hand, sync it from upstream. `CLAUDE.md`, `wiki/`, `raw/`, and `.instructions/projects/` are yours.

---

## Quick start

### 1. Use the template

Click the green **Use this template** button on [github.com/BFreeLabs/karpathy-wiki](https://github.com/BFreeLabs/karpathy-wiki), or:

```bash
gh repo create my-second-brain --template BFreeLabs/karpathy-wiki --public
gh repo clone my-second-brain
cd my-second-brain
```

### 2. Open in Claude Code

Open the folder in [Claude Code](https://claude.ai/code) (and optionally [Obsidian](https://obsidian.md) for human browsing).

### 3. Customize for your topic

In Claude Code, type:

> *"run the customize-template prompt"*

Claude reads `.instructions/core/prompts/customize-template.md` and walks you through 4 questions:

1. **Topic scope** — what's this wiki about? (One paragraph.)
2. **Domain tags** — your 5–10 reusable tags
3. **Project-specific use cases** — anything beyond standard ingest? (Recipe testing, hardware benchmarks, paper reading lists, etc.)
4. **Branding** — name, URL, social handles (optional)

It pauses for your approval, then in a single commit updates `CLAUDE.md`, `README.md`, the wiki state files, and any project scaffolding.

### 4. Add your first source

Drop an article, transcript, or PDF into `raw/`. (Obsidian Web Clipper makes this trivial.) Then tell Claude:

> *"run the ingest prompt"*

Claude reads the source, proposes a Phase 1 plan, waits for your approval, then writes the source summary, creates or updates entity/concept pages, cross-references everything, updates `index.md` and `hot.md`, archives the source, and commits.

### 5. Publish (optional)

See [GitHub Pages setup](#github-pages-setup) below.

---

## Using a different agent (Codex, Cursor, Aider, Continue, etc.)

This template was built around [Claude Code](https://claude.ai/code), but **nothing in `.instructions/core/` is Claude-specific**. The prompts, rules, and templates are plain markdown — any coding agent that can read files and run commands can drive this wiki. The only thing that's Claude-specific is the file *name* `CLAUDE.md`, which Claude Code auto-loads on every session.

To use a different agent, do **one** of the following:

### Option A — Rename `CLAUDE.md` to whatever your agent auto-loads

Most modern coding agents have an auto-loaded project instructions file. Pick the right one for your tool:


| Agent                   | Auto-loaded file                                                              | Action                                            |
| ------------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------- |
| **Claude Code**         | `CLAUDE.md`                                                                   | Leave as-is                                       |
| **OpenAI Codex CLI**    | `AGENTS.md`                                                                   | `mv CLAUDE.md AGENTS.md`                          |
| **Cursor**              | `.cursorrules` or `.cursor/rules/*.mdc`                                       | `mv CLAUDE.md .cursorrules` (or split into rules) |
| **Aider**               | `CONVENTIONS.md` (loaded via `--read CONVENTIONS.md` or in `.aider.conf.yml`) | `mv CLAUDE.md CONVENTIONS.md`                     |
| **Continue**            | `.continue/rules/*.md`                                                        | Move to`.continue/rules/wiki.md`                  |
| **GitHub Copilot Chat** | `.github/copilot-instructions.md`                                             | `mv CLAUDE.md .github/copilot-instructions.md`    |
| **Windsurf**            | `.windsurfrules`                                                              | `mv CLAUDE.md .windsurfrules`                     |

After renaming, also update the small handful of cross-references in `README.md` and `.instructions/core/prompts/*.md` that mention "CLAUDE.md" by name — or just ask your agent to do it: *"Rename CLAUDE.md to AGENTS.md and update every reference."*

### Option B — Keep `CLAUDE.md` and point your agent at it manually

If your agent doesn't auto-load any file, just tell it where the project rules live at the start of every session:

> *"Read CLAUDE.md before doing anything else. It defines this project's scope, rules, and which prompts to invoke."*

This works with any LLM that can read files (ChatGPT with file access, Gemini, local Ollama setups via `aider --read CLAUDE.md`, etc.). It's slightly less ergonomic than auto-loading but fully functional.

### Why the methodology is agent-agnostic

The whole design assumes the agent's job is to *read a prompt file and follow its protocol*, not to remember rules from training. So as long as your agent can:

1. Read markdown files on demand
2. Edit files
3. Run shell commands (`git`, `gh`, optional `pypdf`)

…it can drive this template. The author tests primarily on Claude Code, but PRs improving compatibility notes for other agents are welcome.

---

## Best practices: tell the agent to customize itself

The biggest mistake new users make is treating `.instructions/core/` as something to edit by hand. **Don't.** Instead, when something feels off, *describe the problem to Claude and ask it to propose a methodology change*. Examples:

- *"The lint prompt keeps flagging code-block wikilinks as dangling. Propose a fix to lint.md."*
- *"My `tasks.md` is becoming a dumping ground. What rule would prevent that?"*
- *"I want a new page type for `recipe-test` with its own template. Walk me through adding it."*
- *"My `hot.md` keeps growing past 500 words. Why isn't the lint catching it? Propose a fix."*

For each of these the agent should:

1. Read the relevant rule/prompt file
2. Propose a concrete change (you approve it)
3. Apply it to the local `.instructions/core/` file
4. **Suggest you open a PR upstream** so the improvement benefits everyone

This is the loop. The methodology evolves through usage, not through hand-editing.

### Other good habits

- **Run `lint` after every batch of ingests.** Catches duplicates, dangling links, frontmatter drift, size-budget violations.
- **Run `task-review` weekly** instead of letting `tasks.md` accumulate.
- **Run `upgrade-from-template` monthly** to pull methodology improvements (see below).
- **Never edit `wiki/log.md` by hand.** The agent rewrites it under a 10-entry rolling cap. Git is the real history.
- **Keep your `CLAUDE.md` Wiki Scope tight.** One paragraph. The agent reads it on every session start.

---

## Upgrading the methodology over time

The methodology under `.instructions/core/` is versioned in this template repo. As bugs get fixed and rules get sharper, you'll want to pull those improvements into your wiki **without losing your own content**.

In Claude Code, type:

> *"run the upgrade-from-template prompt"*

Claude will:

1. Add this template repo as a `template` git remote (one time)
2. `git fetch template`
3. Diff `.instructions/core/` between your wiki and `template/main`
4. Present a structured report of new files, modified files, and required `CLAUDE.md` table updates
5. **Pause for your approval** of which items to apply
6. Apply only the approved subset (never touching `wiki/`, `raw/`, `CLAUDE.md` Wiki Scope, or `.instructions/projects/`)
7. Append a single entry to `wiki/log.md` and propose a commit

Full protocol in [`.instructions/core/prompts/upgrade-from-template.md`](.instructions/core/prompts/upgrade-from-template.md).

---

## GitHub Pages setup

This template ships with a [Quartz v4](https://quartz.jzhao.xyz/) publish workflow ([`.github/workflows/publish.yml`](.github/workflows/publish.yml)) **enabled by default**. It builds your `wiki/` directory and deploys to GitHub Pages on:

- Version tags (`v*`)
- Manual workflow dispatch
- Nightly at midnight UTC

### Don't want to publish? Disable it

If you'd rather keep your wiki private (or just not burn Actions minutes until you're ready), rename the workflow file:

```bash
mv .github/workflows/publish.yml .github/workflows/publish.yml.disabled
git add .github/workflows/
git commit -m "chore: disable publish workflow"
git push
```

GitHub Actions only picks up files matching `*.yml` / `*.yaml`, so the renamed file is fully inert. Rename it back when you're ready to publish.

### Enable GitHub Pages on your repo

1. Push at least one commit to `main`
2. Go to your repo → **Settings → Pages**
3. Under **Build and deployment → Source**, choose **GitHub Actions**
4. (Optional) Add a custom domain in the same panel; create a `CNAME` file at the repo root if you do

### Configure Quartz

Edit [`quartz.config.ts`](quartz.config.ts):

```ts
configuration: {
  pageTitle: "Your Wiki Name",
  baseUrl: "your-site.example.com",   // no protocol, no trailing slash
  // ...
}
```

### Trigger the first build

```bash
git tag v$(date +%Y.%m.%d)
git push --tags
```

Watch the run in your repo's **Actions** tab. After it succeeds, your site is live at the URL shown in the deploy step.

### Build locally

```bash
bash .github/scripts/build.sh         # builds wiki/ → /tmp/quartz-output
node .github/scripts/serve.js         # serves at http://localhost:8000
```

---

## Daily workflow

1. Clip an article (Obsidian Web Clipper, save-as-markdown, paste-and-save) into `raw/`
2. Tell Claude: *"run the ingest prompt"*
3. Approve the Phase 1 plan
4. Claude writes everything, commits, and you push
5. The Action rebuilds and deploys overnight (or tag a release for an immediate build)

That's it. The wiki grows itself.

---

## Contributing methodology back upstream

Every time you find a methodology improvement — a sharper lint rule, a new page template, a fixed prompt — please open a PR against [BFreeLabs/karpathy-wiki](https://github.com/BFreeLabs/karpathy-wiki) touching only files under `.instructions/core/`. Topic-specific or wiki-specific changes will be rejected; that's the whole point of the `core/` boundary.

See [issue templates](.github/ISSUE_TEMPLATE) for the three accepted issue types: bug report, feature request, and methodology change.

---

## Credits

- **Pattern**: [Andrej Karpathy's LLM Wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), demonstrated by [Nate Herk](https://www.youtube.com/watch?v=sboNwYmH3AY)
- **Build system**: [Quartz v4](https://quartz.jzhao.xyz/) by jackyzha0
- **Content tool**: [Claude Code](https://claude.ai/code)
- **Editor**: [Obsidian](https://obsidian.md) (optional but recommended)
- **PDF extraction**: [pypdf](https://github.com/py-pdf/pypdf)

---

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, customize it, share it.
