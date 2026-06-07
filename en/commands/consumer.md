---
layout: default
title: consumer Command
parent: Command Reference
nav_order: 20
---

# consumer Command
{: .no_toc }

Manage result consumers. A consumer creates a bounded buffer for job, probe, or target output so CLI, TUI, MCP, or API clients can read diagnostic result streams.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Syntax

```bash
peeka-cli consumer <subcommand> [options]
```

## Subcommands

| Subcommand | Description |
|------------|-------------|
| `create` | Create a result consumer |
| `list` | List result consumers |
| `status --consumer <id>` | Show consumer status |
| `drain --consumer <id>` | Read buffered records |
| `close --consumer <id>` | Close a consumer |
| `cleanup` | Clean closed or failed consumers |

## create Options

| Option | Description |
|--------|-------------|
| `--target <id>` | Owning target |
| `--source cli/tui/mcp/api/internal` | Request source |
| `--scope-type job/probe/target` | Scope type to consume |
| `--scope-id <id>` | Job, probe, or target ID |
| `--client <id>` | Optional owning client |
| `--max-buffer-size <n>` | Maximum buffered records, default `1000` |
| `--backpressure-policy drop_oldest/drop_newest/fail` | Policy when the buffer is full, default `drop_oldest` |

## drain Options

| Option | Description |
|--------|-------------|
| `--limit <n>` | Maximum records to return, default `100` |
| `--after-sequence <n>` | Return records with sequence greater than this value |
| `--timeout-ms <n>` | Wait up to this many milliseconds for new records, default `0` |

All subcommands support `--format table` or `--format json`.

## Examples

```bash
peeka-cli consumer create \
  --target target_abcd1234 \
  --source cli \
  --scope-type probe \
  --scope-id probe_123 \
  --format json

peeka-cli consumer drain --consumer consumer_123 --limit 50 --format json
peeka-cli consumer close --consumer consumer_123
peeka-cli consumer cleanup --target target_abcd1234
```

## Version History

| Version | Release Date | Changes |
|---------|--------------|---------|
| 0.1.16 | 2026-06-07 | Added the `consumer` command group |

## Related Commands

- [client]({% link commands/client.md %}) - Manage client sessions
- [job]({% link commands/job.md %}) - Manage command jobs
- [probe]({% link commands/probe.md %}) - Manage probe runs
