---
layout: default
title: Installation Guide
nav_order: 2
permalink: /installation
---

# Installation Guide
{: .no_toc }

This guide will help you install and configure Peeka in different environments.
{: .fs-6 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## System Requirements

### Basic Requirements

- **Python Version**: Python 3.8.1 or higher
- **Operating System**: Linux (recommended), macOS
- **Permissions**: Permission to attach to target processes (same UID or CAP_SYS_PTRACE)

### Python Version Comparison

| Python Version | Attach Mechanism | Additional Requirements |
|------------|---------|---------|
| **3.14+** | PEP 768 `sys.remote_exec` | None |
| **3.8.1-3.13** | Linux: GDB + ptrace; macOS: LLDB + dlopen | Linux: GDB 7.3+, python3-dbg, CAP_SYS_PTRACE; macOS: Xcode Command Line Tools |

---

## Installation Methods

### Install with pip (Recommended)

#### Basic Version (CLI Only)

```bash
pip install peeka
```

#### Full Version (with TUI)

```bash
pip install peeka[tui]
```
### Install with uv

```bash
# Basic version
uv pip install peeka

# Full version (with TUI)
uv pip install "peeka[tui]"

# Development environment (from source)
uv sync --dev
```

### Install from Source

```bash
# Clone the repository
git clone https://github.com/peeka-project/peeka.git
cd peeka

# Install (basic version)
uv pip install -e .

# Install (with TUI)
uv pip install -e ".[tui]"

# Development environment (full dependencies)
uv sync --dev
```

---

## Additional Configuration for Python < 3.14

For Python 3.8.1-3.13, Linux needs GDB and Python debugging symbols; macOS uses LLDB and needs Xcode Command Line Tools.

### Debian/Ubuntu

```bash
sudo apt-get update
sudo apt-get install gdb python3-dbg
```

### RHEL/CentOS/Fedora

```bash
sudo yum install gdb python3-debuginfo
```

### macOS

```bash
xcode-select --install
```

---

## Permission Configuration

### Linux Systems

#### Temporarily Relax ptrace Restrictions (Testing Only)

```bash
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

#### Permanent Configuration (Recommended for Production)

Edit `/etc/sysctl.d/10-ptrace.conf`:

```
kernel.yama.ptrace_scope = 1
```

Then apply the configuration:

```bash
sudo sysctl -p /etc/sysctl.d/10-ptrace.conf
```

#### SELinux Systems (Fedora/RHEL)

```bash
# Check SELinux status
getenforce

# Temporarily allow ptrace
sudo setsebool -P deny_ptrace off

# Or create SELinux policy for specific processes
```

### Docker Containers

Add `--cap-add=SYS_PTRACE` parameter when running Docker containers:

```bash
docker run --cap-add=SYS_PTRACE your-image
```

Or configure in docker-compose.yml:

```yaml
services:
  app:
    image: your-image
    cap_add:
      - SYS_PTRACE
    security_opt:
      - seccomp=unconfined
```

---

## Verify Installation

### Check Version

```bash
peeka-cli --version
```

### Run Tests

```bash
# Start demo application
python -m peeka.examples.demo --mode loop

# Test attachment in another terminal
peeka-cli attach <pid>
```

### Check Dependencies

```bash
# Check Python version
python --version

# Check GDB (Linux, Python < 3.14)
gdb --version

# Check LLDB (macOS, Python < 3.14)
lldb --version

# Check Python debugging symbols (Linux, Python < 3.14)
python -c "import sys; print(hasattr(sys, 'gettotalrefcount'))"
```

---

## Common Issues

### Attachment Failed: Insufficient Permissions

**Error Message**:
```
Error: Operation not permitted
```

**Solution**:
1. Ensure you have the same UID as the target process, or use sudo
2. Check ptrace_scope settings
3. For Docker, ensure CAP_SYS_PTRACE is added

### Python < 3.14 on Linux: Debugging Symbols Not Found

**Error Message**:
```
Error: Python debugging symbols not found
```

**Solution**:
```bash
# Debian/Ubuntu
sudo apt-get install python3-dbg

# RHEL/CentOS
sudo yum install python3-debuginfo
```

### GDB Version Too Old

**Error Message**:
```
Error: GDB version 7.3+ required
```

**Solution**:
```bash
# Update GDB
sudo apt-get update
sudo apt-get install --only-upgrade gdb

# Or compile the latest version from source
```

### macOS: LLDB Unavailable or Permission Denied

**Error Message**:
```
Error: LLDB not found
Error: LLDB attach failed: permission denied
```

**Solution**:
Install Xcode Command Line Tools:

```bash
xcode-select --install
```

---

## Next Steps

After installation, you can:

- [Quick Start]({% link quickstart.md %}) - Learn basic usage
- [Command Reference]({% link commands/index.md %}) - View all available commands
- [Examples]({% link examples.md %}) - Real-world application scenarios

---

## Getting Help

If you encounter problems during installation:

1. Check [Troubleshooting]({% link troubleshooting.md %})
2. Ask questions on [GitHub Issues](https://github.com/peeka-project/peeka/issues)
