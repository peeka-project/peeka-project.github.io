---
layout: default
title: probe Command
parent: Command Reference
nav_order: 19
---

# probe Command
{: .no_toc }

Manage probe runs. Observation commands such as `watch`, `trace`, `monitor`, and `top` can register probes inside a target; `probe` lists, inspects, stops, and cleans those runs.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Syntax

```bash
peeka-cli probe <subcommand> [options]
```

## Subcommands

| Subcommand | Description |
|------------|-------------|
| `list` | List probe runs |
| `status --probe <id>` | Show probe status |
| `inspect --probe <id>` | Show probe details and recent events |
| `stop --probe <id>` | Stop a running probe |
| `cleanup` | Clean old probes |

Common options:

| Option | Description |
|--------|-------------|
| `--target <id>` | Filter by or locate the owning target |
| `--type <type>` | Filter `list` by probe type, such as `watch` or `trace` |
| `--status <status>` | Filter `list` by status |
| `--events <n>` | Number of recent events returned by `inspect`, default `100` |
| `--all` | During `cleanup`, also clean created/paused probes; never active probes |
| `--older-than <duration>` | During `cleanup`, clean probes older than the duration; default `10m` |
| `--format table/json` | Output format, default `table` |

## Examples

```bash
peeka-cli probe list --target target_abcd1234
peeka-cli probe list --type watch --status active
peeka-cli probe inspect --probe probe_123 --events 20 --format json
peeka-cli probe stop --probe probe_123
peeka-cli probe cleanup --older-than 1h
```

## Version History

| Version | Release Date | Changes |
|---------|--------------|---------|
| 0.1.16 | 2026-06-07 | Added the `probe` command group |

## Related Commands

- [watch]({% link commands/watch.md %}) - Create function observations
- [trace]({% link commands/trace.md %}) - Create call-chain traces
- [job]({% link commands/job.md %}) - Manage command jobs
