---
layout: default
title: client Command
parent: Command Reference
nav_order: 17
---

# client Command
{: .no_toc }

Create and manage client sessions bound to target agents. Client sessions distinguish CLI, TUI, MCP, API, or internal callers.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Syntax

```bash
peeka-cli client <subcommand> [options]
```

## Subcommands

| Subcommand | Description |
|------------|-------------|
| `create --target <id> --source <source>` | Create a client session |
| `list [--target <id>]` | List client sessions, optionally filtered by target |
| `status --client <id>` | Show client session status |
| `close --client <id>` | Close a client session |

`--source` can be `cli`, `tui`, `mcp`, `api`, or `internal`. `create` also accepts `--user <id>` for an optional user identifier.

All subcommands support `--format table` (default) or `--format json`.

## Examples

```bash
# Create a client for CLI automation
peeka-cli client create --target target_abcd1234 --source cli --format json

# List clients for one target
peeka-cli client list --target target_abcd1234

# Inspect and close a client
peeka-cli client status --client client_123
peeka-cli client close --client client_123
```

## Typical Uses

- Give each caller a distinct identity when multiple tools connect to the same target.
- Associate job, probe, consumer, or dx operations with a specific client.
- Create a client first and pass `--client` when automation needs explicit ownership or access control.

## Version History

| Version | Release Date | Changes |
|---------|--------------|---------|
| 0.1.16 | 2026-06-07 | Added the `client` command group |

## Related Commands

- [target]({% link commands/target.md %}) - Manage targets
- [job]({% link commands/job.md %}) - Manage command jobs
- [consumer]({% link commands/consumer.md %}) - Manage result consumers
