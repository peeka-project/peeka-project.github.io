# Agent Guidelines

This repository is the GitHub Pages documentation site for Peeka. Keep changes small, consistent, and easy to review.

## Project Shape

- Root docs are Chinese.
- English, Spanish, and Japanese docs live under `en/`, `es/`, and `ja/`.
- Command reference pages live in `commands/` and each language's `commands/` directory.
- The site uses Jekyll with the Just the Docs theme.
- Do not edit generated output in `_site/`, `.jekyll-cache/`, `.sass-cache/`, `.bundle/`, or `vendor/`.

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
- Keep the worktree clean after committing and pushing.
