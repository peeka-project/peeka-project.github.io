---
layout: default
title: job Command
parent: Command Reference
nav_order: 18
---

# job Command
{: .no_toc }

Manage asynchronous command jobs: view lifecycle state, inspect result metadata, interrupt running work, and clean old jobs.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Syntax

```bash
peeka-cli job <subcommand> [options]
```

## Subcommands

| Subcommand | Description |
|------------|-------------|
| `list` | List command jobs |
| `status --job <id>` | Show job status |
| `inspect --job <id>` | Show full job details |
| `interrupt --job <id>` | Interrupt a running job |
| `cleanup` | Clean old jobs |
| `pull --job <id> --consumer <name>` | Reserved result-pull interface; currently a Phase 5 stub |

Common filters and options:

| Option | Description |
|--------|-------------|
| `--target <id>` | Filter by or locate the owning target |
| `--client <id>` | Filter `list` by client session |
| `--status <status>` | Filter `list` by job status |
| `--completed` | During `cleanup`, clean completed jobs only |
| `--older-than <duration>` | During `cleanup`, clean jobs older than the duration; default `10m` |
| `--format table/json` | Output format, default `table` |

`--older-than` accepts seconds or durations such as `30s`, `10m`, or `2h`.

## Examples

```bash
peeka-cli job list --target target_abcd1234
peeka-cli job status --job job_123 --format json
peeka-cli job inspect --job job_123 --target target_abcd1234
peeka-cli job interrupt --job job_123
peeka-cli job cleanup --completed --older-than 30m
```

## Version History

| Version | Release Date | Changes |
|---------|--------------|---------|
| 0.1.16 | 2026-06-07 | Added the `job` command group |

## Related Commands

- [probe]({% link commands/probe.md %}) - Manage probe runs
- [consumer]({% link commands/consumer.md %}) - Read buffered results
- [target]({% link commands/target.md %}) - Select a target
