---
layout: default
title: Home
nav_order: 1
description: "Peeka - Python Runtime Diagnostic Tool with Non-Invasive Function Observation based on PEP 768"
permalink: /
---

# Peeka
{: .fs-9 }

> *Peek-a-boo!* — The name comes from the children's game. A diagnostic tool finding a hidden bug feels just like that moment of surprise when someone is found in hide-and-seek.

A runtime diagnostic tool based on Python 3.14 remote debugging protocol (PEP 768), providing non-invasive function observation capabilities.
{: .fs-6 .fw-300 }

This documentation covers Peeka v{{ site.peeka_version }}.
{: .text-grey-dk-000 }

[Quick Start](#quick-start){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on GitHub](https://github.com/peeka-project/peeka){: .btn .fs-5 .mb-4 .mb-md-0 }

---

{: .note }
> 🌐 **Language / 语言**: This documentation is also available in [Chinese (中文)](/), [Español](/es/), and [日本語](/ja/).
>
> 本文档也提供[中文](/)、[西班牙语](/es/)和[日语](/ja/)版本。

---

## What is Peeka?

Peeka is a tool that provides real-time diagnostic capabilities for Python developers in production environments. It can dynamically observe and diagnose running Python applications **without modifying target code**.

### Why Peeka?

Traditional Python debugging methods face many challenges in production environments:

- **Cannot stop service** - Breakpoint debugging blocks the process
- **Intermittent issues** - Need to observe under real load
- **Code modification difficulties** - Cannot deploy freely in production

Peeka is designed specifically to solve these production environment diagnostic challenges.

---

## Core Features

### 🔍 Non-Invasive Observation
{: .text-delta }

- No need to modify target code
- Runtime dynamic injection of observation logic
- Complete restoration after diagnosis

### ⚡ Real-Time Diagnosis
{: .text-delta }

- Millisecond-level data transmission latency
- Streaming observation data push
- JSON format output for easy integration with other tools

### 🛡️ Production-Ready
{: .text-delta }

- Performance overhead < 5%
- Complete exception capture and recovery mechanisms
- Fixed memory buffer to prevent memory expansion

### 🎯 Conditional Filtering
{: .text-delta }

- Safe expression filtering (based on simpleeval)
- Flexible filtering syntax (parameters, return values, execution time, etc.)
- Blocks all code injection attacks (`__import__`, `eval`, `exec`, etc.)

---

## Quick Start

### Installation

```bash
pip install peeka
```

### Basic Usage

#### 1. Attach to Target Process

```bash
peeka-cli attach <pid>
```

#### 2. Observe Function Calls

```bash
# Observe 5 calls
peeka-cli watch "module.Class.method" --times 5

# Conditional filtering
peeka-cli watch "module.Class.method" --condition "len(params) > 2"

# Real-time streaming observation
peeka-cli watch "module.Class.method"
```

#### 3. Data Processing

```bash
# Extract results with jq
peeka-cli watch "module.func" | jq 'select(.type == "observation") | .data.result'

# Filter slow calls
peeka-cli watch "module.func" | jq 'select(.type == "observation" and .data.duration_ms > 1)'
```

---

## Main Features

| Command | Function | Status |
|---------|----------|--------|
| `attach` | Attach to target process | ✅ |
| `watch` | Observe function calls (params, return, time) | ✅ |
| `trace` | Trace call chain and execution time | ✅ |
| `stack` | Trace call stack | ✅ |
| `monitor` | Performance statistics monitoring | ✅ |
| `logger` | Dynamically adjust log level | ✅ |
| `memory` | Memory analysis | ✅ |
| `inspect` | Runtime object inspection | ✅ |
| `sc/sm` | Search classes and methods | ✅ |
| `reset` | Reset enhancements | ✅ |
| `thread` | Thread analysis | ✅ |
| `top` | Function-level performance sampling | ✅ |
| `patch-status` | Runtime patch and RPL integrity diagnostics | ✅ |
| `detach` | Safely exit diagnostic session | ✅ |

[View Complete Command Reference]({% link commands/index.md %}){: .btn .btn-outline }

### 🎨 Interactive TUI Interface
{: .text-delta }

In addition to CLI commands, Peeka also provides a feature-complete TUI (Text User Interface):

- **Process Selector** - Automatically displays system process list with search filtering and attach progress/log panels
- **10 Dedicated Views** - Dashboard, Watch, Trace, Stack, Monitor, Logger, Memory, Inspect, Threads, Top
- **Real-Time Data Stream** - Streaming observation data with pause/resume/clear support
- **Auto-Completion** - Dynamically retrieve classes and methods from target process
- **Theme Support** - Multiple built-in color themes

![Peeka Dashboard view]({{ site.url }}/assets/images/screenshots/peeka-dashboard.png)

```bash
# Launch TUI
peeka

# Use number keys to switch views
# 1/2/3/4/5/6/7/8/9/0
```

[View Complete TUI Usage Guide]({% link tui.md %}){: .btn .btn-outline }

---

---

## Technical Highlights

### Python 3.14 Remote Debugging Protocol (PEP 768)

The core `sys.remote_exec(pid, script_path)` function is the key to the entire system's operation, encapsulating complex process attachment, code injection, and execution scheduling logic.

**Fallback for Python < 3.14**: For older Python versions, uses GDB + ptrace on Linux and LLDB + dlopen on macOS (reference: pyrasite).

### Unix Domain Socket

Uses Unix Domain Socket as the main inter-process communication mechanism:

- Higher transmission efficiency - No need for network protocol stack
- Stronger security - Only local processes can use
- Simple and reliable - Length prefix + JSON format

### Safe Conditional Expression Evaluation

Implements safe conditional filtering based on simpleeval library:

- AST whitelist - Only allows safe operations
- Attribute protection - Blocks reflection attacks
- Function blacklist - Disables dangerous functions

[Learn Architecture Design]({% link architecture.md %}){: .btn .btn-outline }

---

## Supported Python Versions

| Python Version | Attachment Mechanism | Requirements |
|----------------|---------------------|--------------|
| 3.14+ | PEP 768 `sys.remote_exec` | None |
| 3.8.1-3.13 | Linux: GDB + ptrace; macOS: LLDB + dlopen | Linux: GDB 7.3+, python3-dbg, CAP_SYS_PTRACE; macOS: Xcode Command Line Tools |

---

## License

Peeka is open source under [Apache License 2.0](https://github.com/peeka-project/peeka/blob/main/LICENSE).

---

## Acknowledgments

- Security evaluation: [simpleeval](https://github.com/danthedeckie/simpleeval)
- Security evaluation: [simpleeval](https://github.com/danthedeckie/simpleeval)
- Remote debugging protocol: [PEP 768](https://peps.python.org/pep-0768/)
