---
layout: default
title: target Command
parent: Command Reference
nav_order: 16
---

# target Command
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

The `target` command **manages Peeka target agents**: discover Peeka targets on the current host, view status, clean up stale markers, and detach targets by ID. It is the gateway for all other diagnostic commands - only when a target is confirmed `alive` can `watch`, `trace`, `inspect`, and friends be routed to the right process.

Use `list`, `current`, `status`, `inspect`, `cleanup`, and `detach` to pinpoint the right target when several coexist on the same host, and to promptly reclaim stale targets whose socket markers were left behind after the process exited.

## Syntax

```bash
peeka-cli target <subcommand> [options]
```

`target` operates on processes injected or managed by Peeka. Target states include `alive`, `stale`, `unknown`, `attaching`, `failed`, and `detached`.

`session` remains as a deprecated compatibility alias:

```bash
peeka-cli session list
```

Use `peeka-cli target ...` for new scripts.

## Subcommands

| Subcommand | Description |
|------------|-------------|
| `list` | List discovered target agents |
| `current` | Return the current target when exactly one target is `alive` |
| `status --target <id>` | Show a target status summary |
| `inspect --target <id>` | Show full target details, capabilities, and next actions |
| `cleanup [--target <id>] [--dry-run]` | Clean stale target markers; stale-only cleanup is the default |
| `detach --target <id> [--force]` | Detach a target; alive targets require `--force` |

All subcommands support:

| Option | Description |
|--------|-------------|
| `--format table` | Default table output |
| `--format json` | JSONL events for scripts |

## Examples

```bash
# List all targets
peeka-cli target list

# Return the target when there is exactly one alive target
peeka-cli target current

# Inspect details
peeka-cli target inspect --target target_abcd1234 --format json

# Preview stale marker cleanup
peeka-cli target cleanup --dry-run

# Detach an alive target
peeka-cli target detach --target target_abcd1234 --force
```

## current Exit Codes

| Exit Code | Meaning |
|-----------|---------|
| `0` | Exactly one alive target exists |
| `1` | No alive target exists |
| `2` | Multiple alive targets exist; choose explicitly |

## Version History

| Version | Release Date | Changes |
|---------|--------------|---------|
| 0.1.16 | 2026-06-07 | Added the `target` command group; kept `session` as a deprecated compatibility alias |

## Related Commands

- [attach]({% link commands/attach.md %}) - Attach to a target process
- [detach]({% link commands/detach.md %}) - Detach the current session
- [client]({% link commands/client.md %}) - Manage client sessions
