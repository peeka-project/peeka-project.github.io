# Release Documentation Sync

Usage: `/release <version>`

Treat `$ARGUMENTS` as `<version>`. It is a Peeka source git tag version. Accept
either `0.1.13` or `v0.1.13`; normalize it to a tag named `v0.1.13` before
running source git commands.

## Goal

Compare the currently documented Peeka source tag with the requested target
source tag, decide whether the documentation site needs updates, and either:

- update the documentation according to a concrete plan, or
- explain briefly why no documentation changes are needed.

The Peeka source checkout is provided by this repository's `AGENTS.md` and
`.env`. Do not guess another path.

The documentation repository itself does not need release tags for this
workflow. "Currently documented tag" means the Peeka source tag recorded in the
documentation content, especially command-page Version History tables and other
explicit version notes.

## Workflow

1. Load the source path from `.env`:

   ```bash
   set -a; . ./.env; set +a
   ```

   Use `$PEEKA_SRC` for all source-repository git commands. If the path is
   missing or invalid, follow `AGENTS.md` before proceeding.

2. Validate the argument:

   - If no version argument is provided, ask for one.
   - Prefix the argument with `v` when it does not already start with `v`.
   - Confirm the normalized target tag exists:

     ```bash
     git -C "$PEEKA_SRC" rev-parse --verify --quiet "<target-tag>^{commit}"
     ```

3. Find the currently documented source tag from existing documentation
   content. Do not use git tags from this documentation repository. Prefer the
   highest semver tag mentioned under command docs:

   ```bash
   rg -o 'v0\.[0-9]+\.[0-9]+' commands en/commands es/commands ja/commands | sort -V | tail
   ```

   If command docs do not contain a newer tag but another source-controlled docs
   page explicitly documents one, treat that as evidence and explain the choice.
   If the docs contain inconsistent latest versions across languages, stop and
   report the inconsistency before editing.

4. Compare the documented tag with the target tag:

   ```bash
   git -C "$PEEKA_SRC" log --oneline <documented-tag>..<target-tag>
   git -C "$PEEKA_SRC" diff --stat <documented-tag>..<target-tag>
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

   - the documented tag,
   - the target tag,
   - the inspected commit range summary,
   - the reason the changes do not affect documentation.

7. If docs need updating, first write a concise update plan. Include:

   - affected behavior,
   - source evidence using paths and line numbers, such as
     `../peeka/peeka/cli/main.py:L123`,
   - documentation files to edit in all affected languages,
   - build commands to run.

8. Execute the plan immediately unless the user asked only for a plan.

   Keep all four language variants in lockstep for shared behavior:

   - Chinese root docs
   - English docs under `en/`
   - Spanish docs under `es/`
   - Japanese docs under `ja/`

   For command reference changes, append a row to each affected page's existing
   Version History table. Use the target tag commit date:

   ```bash
   git -C "$PEEKA_SRC" log -1 --format=%ad --date=short <target-tag>
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

10. Commit completed documentation changes with a concise semantic message.
    Do not push.

## Reporting

In the final response, include:

- documented tag and target tag,
- whether docs changed,
- files updated,
- verification result,
- commit hash when a commit was created.
