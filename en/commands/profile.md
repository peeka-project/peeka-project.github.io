---
layout: default
title: profile Command
parent: Command Reference
nav_order: 22
---

# profile Command
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

The `profile` command **creates, inspects, and exports profiling sessions** (ProfileSession) in the target process on demand. Unlike the live, scrolling [`top`]({% link commands/top.md %}), a profile session keeps its samples in the agent, so you can take a snapshot at any time or export it as JSON, text, [speedscope](https://www.speedscope.app/), or [Perfetto](https://ui.perfetto.dev/) for offline analysis and sharing.

`top` itself samples into a transient ProfileSession. It is cleaned up when `top` finishes by default; use `top --keep-profile` to keep it and export it with `profile`.

## Syntax

```bash
peeka-cli attach <pid>    # Attach to the target process first
peeka-cli profile <subcommand> [options]
```

## Subcommands

| Subcommand | Description |
|------------|-------------|
| `start` | Start a profiling session and return its `profile_id` |
| `stop --profile-id <id>` | Stop a session and keep the collected data |
| `snapshot --profile-id <id>` | Take a snapshot of the current data |
| `list` | List all profile sessions |
| `cleanup` | Remove old sessions |
| `export --profile-id <id>` | Export session data, optionally to a file |

### start Options

| Option | Description | Default |
|--------|-------------|---------|
| `--backend sampling/exact` | Profiling backend | `sampling` |
| `--experimental` | Allow the experimental `exact` backend | off |
| `--interval <seconds>` | Sampling interval | `0.01` |
| `--duration <seconds>` | Optional duration; the session stops automatically when it expires | unlimited |
| `--max-samples <N>` | Optional sample cap; the session stops automatically when reached | `100000` |

### snapshot / export Options

| Option | Description | Default |
|--------|-------------|---------|
| `--profile-id <id>` | Session ID (required) | - |
| `--format json/text/speedscope/perfetto` | Output format | `json` |
| `--output <path>` | `export` only: write the exported content to this file | stdout |

### cleanup Options

| Option | Description |
|--------|-------------|
| `--older-than <seconds>` | Remove sessions created more than N seconds ago |
| `--max-sessions <N>` | Keep at most N sessions |

## Backends

- **sampling (default)**: a background thread samples the call stacks of all threads every `--interval`. Low overhead, suitable for long runs on production processes.
- **exact (experimental)**: records every function call from Python call events. Requires Python 3.12+ (`threading.setprofile_all_threads`), has noticeably higher overhead, and must be started with `--experimental`. The event count is capped; the session ends automatically when the cap is reached.

`exact` and `sampling` are mutually exclusive: while one runs, the other cannot start, so the exact hook's overhead cannot skew sampling results. If the target process already has its own profiler installed (`sys.setprofile` / `threading.setprofile`), `exact` refuses to start instead of replacing it.

## Examples

```bash
# Sample for 30 seconds
peeka-cli profile start --duration 30

# List sessions
peeka-cli profile list

# Export as speedscope and open it at https://www.speedscope.app/
peeka-cli profile export --profile-id <profile_id> --format speedscope --output ./profile.speedscope.json

# Stop and clean up
peeka-cli profile stop --profile-id <profile_id>
peeka-cli profile cleanup --max-sessions 5
```

Attach a profile session to a DX case:

```bash
peeka-cli dx add-profile --dx-case dx_123 --profile-id <profile_id>
```

## TUI Usage

In the TUI, press `0` to open the **Performance** tab and switch to the **Profiles** sub-tab to see the profile session list (`profile_id`, `backend`, `status`, `started_at`, `sample_count`). Select a row and click **Export** to export it; press `r` to refresh. See the [TUI Guide]({% link tui.md %}).

## Notes

- Profile sessions live in the target process's agent. Up to 50 sessions are kept; the oldest are evicted beyond that. `detach` stops all active profiles.
- The `speedscope` export produces a separate profile per thread.
- Prefer the default `sampling` backend in production.

## Version History

| Version | Release Date | Changes |
|---------|--------------|---------|
| 0.1.21 | 2026-11-09 | Added the `profile` command group (start / stop / snapshot / list / cleanup / export) with sampling and experimental exact backends and JSON / text / speedscope / Perfetto formats |

## Related Commands

- [top]({% link commands/top.md %}) - Live function performance sampling
- [dx]({% link commands/dx.md %}) - Diagnostic case bundles
- [detach]({% link commands/detach.md %}) - Disconnect and clean up resources
