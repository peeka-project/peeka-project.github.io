---
layout: default
title: patch-status Command
parent: Command Reference
nav_order: 15
---

# patch-status Command
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

`patch-status` is a read-only diagnostic command for checking whether the target process is monkey-patched by gevent/eventlet, whether socket/thread/time stdlib primitives still match the native implementations captured by the Runtime Primitive Layer (RPL), and what asyncio loop and thread model are active.

The command does not remediate or undo monkey patches. It reports state so you can reason about attach, watch, trace, and other diagnostics in complex runtime environments.

## Syntax

```bash
peeka-cli attach <pid>
peeka-cli patch-status [--pid <pid>]
```

| Parameter | Description |
|-----------|-------------|
| `--pid` | Optional compatibility parameter; the current implementation ignores it and reports the active attached session |

You must attach first. If there is no active session, the command returns an error that asks you to run `peeka-cli attach <pid>`.

## Output Fields

On success, `patch-status` returns a `result` message whose nested data contains:

| Field | Description |
|-------|-------------|
| `schema_version` | Output schema version, currently `"1"` |
| `pid` | Target process PID |
| `timestamp` | Sample timestamp |
| `monkey_patch` | gevent/eventlet import and active status, plus patched module lists when available |
| `stdlib_origin` | Current stdlib primitive IDs compared with RPL-captured native IDs |
| `asyncio_loop` | Running loop status, policy, and loop class |
| `thread_model` | Main thread, total thread count, daemon thread count, and classification |
| `rpl_integrity` | Whether captured RPL primitives are intact and whether drift is detected |

## Example

```bash
peeka-cli attach 12345
peeka-cli patch-status | jq '.data.data'
```

Example output:

```json
{
  "schema_version": "1",
  "pid": 12345,
  "monkey_patch": {
    "gevent": {
      "status": "active",
      "patched_modules": ["socket", "threading"]
    },
    "eventlet": "not_imported"
  },
  "stdlib_origin": {
    "socket.socket": {
      "matches": false
    }
  },
  "asyncio_loop": {
    "running": true,
    "policy": "DefaultEventLoopPolicy",
    "loop_class": "SelectorEventLoop"
  },
  "thread_model": {
    "total_threads": 4,
    "classification": "multi_threaded_with_daemons"
  },
  "rpl_integrity": {
    "ok": true
  }
}
```

## Typical Uses

- Confirm monkey patch state before or after attaching to gevent or eventlet services
- Investigate attach or observation issues caused by replaced socket, threading, or time primitives
- Verify that RPL still has native socket/thread/time primitives available for Peeka internals
- Check whether the target process currently runs an asyncio loop and whether the thread model is expected

## Interpreting Results

- `monkey_patch.gevent.status == "active"` or `monkey_patch.eventlet.status == "active"` means that runtime monkey patching is active.
- `stdlib_origin.*.matches == false` means the current stdlib object differs from the native object captured by RPL.
- `rpl_integrity.ok == true` means RPL's captured native primitives are still available for Peeka's internal diagnostic path.
- `patch-status` does not restore native functions. To remove Peeka observation enhancements, use [reset]({% link commands/reset.md %}) or [detach]({% link commands/detach.md %}).

## Version History

| Version | Release Date | Changes |
|---------|--------------|---------|
| 0.1.14 | 2026-05-24 | Added the `patch-status` runtime diagnostic command |

## Related Commands

- [attach]({% link commands/attach.md %}) - Attach to target process
- [watch]({% link commands/watch.md %}) - Observe function calls
- [trace]({% link commands/trace.md %}) - Trace call chains
- [reset]({% link commands/reset.md %}) - Reset enhancements
