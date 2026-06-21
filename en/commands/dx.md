---
layout: default
title: dx Command
parent: Command Reference
nav_order: 21
---

# dx Command
{: .no_toc }

Create, organize, and export diagnostic case bundles. A DX case collects target, client, job, probe, consumer, error, note, and summary sections into one exportable context.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

The `dx` command **creates, organizes, and exports diagnostic case bundles** (DX cases): it gathers target, client, job, probe, consumer, errors, notes, and summaries into a single exportable context, making post-mortem analysis and team collaboration straightforward.

Use `create`, `add`, `summary`, and `export` to consolidate every clue from a diagnostic session into a self-contained artifact instead of letting it scatter across terminals and sessions. The exported DX case is also easy to attach to issues or PRs as a reproducible piece of evidence.

## Syntax

```bash
peeka-cli dx <subcommand> [options]
```

## Subcommands

| Subcommand | Description |
|------------|-------------|
| `create --target <id> --title <text>` | Create a DX case |
| `list` | List DX cases |
| `status --dx-case <id>` | Show case status |
| `add --dx-case <id>` | Add a section |
| `summary --dx-case <id>` | Build a summary |
| `export --dx-case <id>` | Export a DX case |
| `close --dx-case <id>` | Close a DX case |

Common options:

| Option | Description |
|--------|-------------|
| `--target <id>` | Owning target; required for `create`, optional for other subcommands |
| `--client <id>` | Optional owning client for access control |
| `--format table/json` | Output format, default `table` |
| `--output-path <path>` | Optional destination path for `export` |

## Add Sections

```bash
peeka-cli dx add \
  --dx-case dx_123 \
  --section-type note \
  --title "Investigation note" \
  --payload-json '{"text":"slow query reproduced"}'
```

`--section-type` accepts `target`, `client`, `job`, `probe`, `consumer`, `note`, `error`, and `summary`. Use `--object-ref-type` and `--object-ref-id` to link an existing object.

## Examples

```bash
peeka-cli dx create --target target_abcd1234 --title "Slow request"
peeka-cli dx list --target target_abcd1234
peeka-cli dx summary --dx-case dx_123
peeka-cli dx export --dx-case dx_123 --output-path ./slow-request.dx.json
peeka-cli dx close --dx-case dx_123
```

## Version History

| Version | Release Date | Changes |
|---------|--------------|---------|
| 0.1.16 | 2026-06-07 | Added the `dx` command group |

## Related Commands

- [target]({% link commands/target.md %}) - Manage targets
- [job]({% link commands/job.md %}) - Collect command job information
- [probe]({% link commands/probe.md %}) - Collect probe events
