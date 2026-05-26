---
name: release
description: Sync this Peeka documentation site to a requested Peeka source release tag. Use when the user invokes `$release VERSION` or asks to compare the currently documented Peeka source tag with a target version, decide whether docs need updates, update docs across all languages, refresh displayed Peeka version metadata, verify Jekyll builds, commit locally, and create the documentation repository tag.
---

# Release Documentation Sync

## Overview

Use `$release <version>` to audit and sync this documentation repository against
a Peeka source tag. Treat the argument as a Peeka source version; accept either
`0.1.13` or `v0.1.13`.

- Use the no-prefix form, such as `0.1.13`, for documentation repository tags.
- Use the `v`-prefixed form, such as `v0.1.13`, for Peeka source repository tags.
- Do not push commits or tags.

The Peeka source checkout is provided by this repository's `AGENTS.md` and
`.env`. Do not guess another path.

The documentation repository tag is the source of truth for the currently
tracked Peeka source version. For example, documentation repository tag
`0.1.13` means the docs currently track Peeka source tag `v0.1.13`.

## Workflow

1. Load the source path from `.env`:

   ```bash
   set -a; . ./.env; set +a
   ```

   Use `$PEEKA_SRC` for all source-repository git commands. If the path is
   missing or invalid, follow `AGENTS.md` before proceeding.

2. Validate and normalize the argument:

   - If no version argument is provided, ask for one.
   - Remove a leading `v` to get `<target-doc-tag>`, such as `0.1.13`.
   - Prefix `<target-doc-tag>` with `v` to get `<target-source-tag>`, such as
     `v0.1.13`.
   - Confirm the target Peeka source tag exists:

     ```bash
     git -C "$PEEKA_SRC" rev-parse --verify --quiet "<target-source-tag>^{commit}"
     ```

   - If the documentation repository already has `<target-doc-tag>`, report
     that the target version is already tracked unless the user explicitly asks
     to audit it again.

3. Find the currently tracked documentation tag from this documentation
   repository's git tags. Prefer the highest semver documentation tag:

   ```bash
   git tag --list '0.*' --sort=-v:refname | head -1
   ```

   Map that documentation tag to the current Peeka source tag by prefixing `v`.
   If no semver documentation tag exists, infer the baseline once from explicit
   version notes in source-controlled documentation content, explain the
   inference, and create the missing baseline documentation tag before using
   this workflow for future releases.

4. Compare the current source tag with the target source tag:

   ```bash
   git -C "$PEEKA_SRC" log --oneline <current-source-tag>..<target-source-tag>
   git -C "$PEEKA_SRC" diff --stat <current-source-tag>..<target-source-tag>
   ```

   Inspect relevant diffs, especially:

   - `pyproject.toml`
   - `peeka/cli/`
   - `peeka/core/`
   - `peeka/tui/`

5. Decide whether documentation changes are needed.

   Documentation updates are needed for user-visible behavior, including CLI
   options, commands, install requirements, supported Python versions, TUI key
   mappings, workflows, examples, troubleshooting behavior, or output that the
   docs describe.

   Documentation updates are not needed for pure refactors, tests, CI-only
   changes, formatting, internal-only implementation changes, or changes to
   undocumented behavior.

6. If no docs need updating, reply with:

   - the current documentation tag and mapped source tag,
   - the target documentation tag and mapped source tag,
   - the inspected commit range summary,
   - the reason the changes do not affect documentation.

   Then create `<target-doc-tag>` on the current documentation commit to record
   that the docs have been audited for the target Peeka source version.

7. If docs need updating, first write a concise update plan. Include:

   - affected behavior,
   - source evidence using paths and line numbers, such as
     `../peeka/peeka/cli/main.py:L123`,
   - documentation files to edit in all affected languages,
   - version metadata files to update,
   - build commands to run.

8. Execute the plan immediately unless the user asked only for a plan.

   Keep all four language variants in lockstep for shared behavior:

   - Chinese root docs
   - English docs under `en/`
   - Spanish docs under `es/`
   - Japanese docs under `ja/`

   Always update the displayed Peeka version metadata to `<target-doc-tag>` in
   all four Jekyll config files, even when no page text changes are otherwise
   required:

   - `_config.yml`
   - `_config_en.yml`
   - `_config_es.yml`
   - `_config_ja.yml`

   These files define `site.peeka_version`, which is rendered on index and
   installation pages. Leaving them stale causes the published site to show the
   old tracked Peeka version even after a successful deployment.

   For command reference changes, append a row to each affected page's existing
   Version History table. Use the target source tag commit date:

   ```bash
   git -C "$PEEKA_SRC" log -1 --format=%ad --date=short <target-source-tag>
   ```

   Do not create a new top-level changelog page.

9. Verify the affected site builds. For broad shared docs changes, run all
   language builds from `AGENTS.md`:

   ```bash
   bundle exec jekyll build --baseurl ""
   bundle exec jekyll build --config _config_en.yml --source en --destination _site/en --baseurl "/en"
   bundle exec jekyll build --config _config_es.yml --source es --destination _site/es --baseurl "/es"
   bundle exec jekyll build --config _config_ja.yml --source ja --destination _site/ja --baseurl "/ja"
   ```

   Existing theme Sass deprecation warnings are acceptable. Liquid errors and
   failed builds are not.

   After building, inspect the generated index and installation pages for all
   languages and confirm they render `<target-doc-tag>` rather than the previous
   version:

   ```bash
   rg '<target-doc-tag>|<previous-doc-tag>' _site/index.html _site/en/index.html _site/es/index.html _site/ja/index.html _site/installation.html _site/en/installation.html _site/es/installation.html _site/ja/installation.html
   ```

10. Commit completed documentation changes with a concise semantic message.
    Then create the documentation repository tag `<target-doc-tag>` on that
    commit. Do not push commits or tags.

## Reporting

In the final response, include:

- documented tag and target tag,
- whether docs changed,
- files updated,
- verification result,
- commit hash when a commit was created,
- documentation tag created.
