# Agent Guidelines

This repository is the GitHub Pages documentation site for Peeka. Keep changes small, consistent, and easy to review.

## Project Shape

- Root docs are Chinese.
- English, Spanish, and Japanese docs live under `en/`, `es/`, and `ja/`.
- Command reference pages live in `commands/` and each language's `commands/` directory.
- The site uses Jekyll with the Just the Docs theme.
- Do not edit generated output in `_site/`, `.jekyll-cache/`, `.sass-cache/`, `.bundle/`, or `vendor/`.

## Source Of Truth

- The local path to the Peeka source repo is recorded in the untracked `.env` file at the repo root as `PEEKA_SRC` (default: `../peeka`). Load it before running git commands against the source: `set -a; . ./.env; set +a`.
- If `$PEEKA_SRC` is missing, clone with `git clone https://github.com/peeka-project/peeka.git "$PEEKA_SRC"`.
- Use source files such as `pyproject.toml`, `peeka/cli/main.py`, and `peeka/tui/screens/main.py` as the reference for supported Python versions, CLI options, command names, and TUI key mappings.

## Iteration Workflow (Tag-Driven Doc Sync)

The normal cadence: a new tag lands in `$PEEKA_SRC`, we walk the commits since the last documented tag and update only the pages whose behavior changed.

1. Find the last documented version (search version-history tables, e.g. `grep -rE 'v0\.1\.[0-9]+' commands/ | sort -V | tail`) and the latest source tag: `git -C "$PEEKA_SRC" tag --sort=-v:refname | head -5`.
2. Diff the range: `git -C "$PEEKA_SRC" log --oneline <last>..<latest>` and `git -C "$PEEKA_SRC" diff --stat <last>..<latest>`. Focus on `peeka/cli/`, `peeka/core/`, `peeka/tui/`, `pyproject.toml`.
3. For each commit, decide: behavior/CLI/TUI/install change → docs update needed; refactor/test/internal → skip.
4. Append a row to the affected command page's existing **Version History** table. Use the tag's commit date: `git -C "$PEEKA_SRC" log -1 --format=%ad --date=short <tag>`. Do NOT create a new top-level changelog page.
5. Cite source as `../peeka/<path>:Lxx` (or SHA) in your reasoning for any new factual claim.
6. Keep all 4 languages in lockstep per topic; never ship a partial-language update.

## Documentation Consistency

- Keep equivalent pages aligned across all languages when changing shared behavior, navigation, command names, shortcuts, or examples.
- Command page `title` and H1 should match:
  - Chinese: `watch 命令`
  - English: `watch Command`
  - Spanish: `Comando watch`
  - Japanese: `watch コマンド`
- Keep TUI key mappings consistent with the current app: `8 = Inspect`, `9 = Threads`, `0 = Top`.
- Prefer updating stale text over adding notes that say docs are incomplete.

## Build And Verify

Run the affected builds before committing. For broad docs changes, run all languages:

```bash
bundle exec jekyll build --baseurl ""
bundle exec jekyll build --config _config_en.yml --source en --destination _site/en --baseurl "/en"
bundle exec jekyll build --config _config_es.yml --source es --destination _site/es --baseurl "/es"
bundle exec jekyll build --config _config_ja.yml --source ja --destination _site/ja --baseurl "/ja"
```

Existing Sass deprecation warnings from the theme are acceptable; Liquid errors or failed builds are not.

## Git

- Use the user's global git config for commits; do not set a bot author.
- Commit completed tasks with concise semantic messages, for example `docs: update command labels`.
- Keep the worktree clean after committing.
- **Do NOT push.** Only commit. Pushing is performed by the user manually.
