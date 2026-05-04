---
layout: default
title: 安装指南
nav_order: 2
---

# 安装指南
{: .no_toc }

本指南将帮助您在不同环境中安装和配置 Peeka。
{: .fs-6 .fw-300 }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 系统要求

### 基本要求

- **Python 版本**: Python 3.8.1 或更高版本
- **操作系统**: Linux（推荐）、macOS
- **权限**: 需要附加到目标进程的权限（相同 UID 或 CAP_SYS_PTRACE）

### Python 版本对比

| Python 版本 | 附加机制 | 额外要求 |
|------------|---------|---------|
| **3.14+** | PEP 768 `sys.remote_exec` | 无 |
| **3.8.1-3.13** | Linux: GDB + ptrace；macOS: LLDB + dlopen | Linux: GDB 7.3+、python3-dbg、CAP_SYS_PTRACE；macOS: Xcode Command Line Tools |

---

## 安装方法

### 使用 pip 安装（推荐）

#### 基础版本（仅 CLI）

```bash
pip install peeka
```

#### 完整版本（包含 TUI）

```bash
pip install peeka[tui]
```

### 使用 uv 安装

```bash
# 基础版本
uv pip install peeka

# 完整版本（包含 TUI）
uv pip install "peeka[tui]"

# 开发环境（从源码）
uv sync --dev
```

### 从源码安装

```bash
# 克隆仓库
git clone https://github.com/peeka-project/peeka.git
cd peeka

# 安装（基础版本）
uv pip install -e .

# 安装（包含 TUI）
uv pip install -e ".[tui]"

# 开发环境（完整依赖）
uv sync --dev
```

---

## Python < 3.14 额外配置

对于 Python 3.8.1-3.13 版本，Linux 需要 GDB 和 Python 调试符号；macOS 使用 LLDB，需要安装 Xcode Command Line Tools。

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

## 权限配置

### Linux 系统

#### 临时放宽 ptrace 限制（仅用于测试）

```bash
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

#### 永久配置（生产环境推荐）

编辑 `/etc/sysctl.d/10-ptrace.conf`:

```
kernel.yama.ptrace_scope = 1
```

然后应用配置：

```bash
sudo sysctl -p /etc/sysctl.d/10-ptrace.conf
```

#### SELinux 系统（Fedora/RHEL）

```bash
# 检查 SELinux 状态
getenforce

# 临时允许 ptrace
sudo setsebool -P deny_ptrace off

# 或者为特定进程创建 SELinux 策略
```

### Docker 容器

在运行 Docker 容器时添加 `--cap-add=SYS_PTRACE` 参数：

```bash
docker run --cap-add=SYS_PTRACE your-image
```

或在 docker-compose.yml 中配置：

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

## 验证安装

### 检查版本

```bash
peeka-cli --version
```

### 运行测试

```bash
# 启动演示应用
python -m peeka.examples.demo --mode loop

# 在另一个终端测试附加
peeka-cli attach <pid>
```

### 检查依赖

```bash
# 检查 Python 版本
python --version

# 检查 GDB（Linux，Python < 3.14）
gdb --version

# 检查 LLDB（macOS，Python < 3.14）
lldb --version

# 检查 Python 调试符号（Linux，Python < 3.14）
python -c "import sys; print(hasattr(sys, 'gettotalrefcount'))"
```

---

## 常见问题

### 附加失败：权限不足

**错误信息**:
```
Error: Operation not permitted
```

**解决方案**:
1. 确保与目标进程有相同的 UID，或使用 sudo
2. 检查 ptrace_scope 设置
3. 对于 Docker，确保添加了 CAP_SYS_PTRACE

### Linux，Python < 3.14：找不到调试符号

**错误信息**:
```
Error: Python debugging symbols not found
```

**解决方案**:
```bash
# Debian/Ubuntu
sudo apt-get install python3-dbg

# RHEL/CentOS
sudo yum install python3-debuginfo
```

### GDB 版本过低

**错误信息**:
```
Error: GDB version 7.3+ required
```

**解决方案**:
```bash
# 更新 GDB
sudo apt-get update
sudo apt-get install --only-upgrade gdb

# 或从源码编译最新版本
```

### macOS: LLDB 不可用或权限不足

**错误信息**:
```
Error: LLDB not found
Error: LLDB attach failed: permission denied
```

**解决方案**:
安装 Xcode Command Line Tools：

```bash
xcode-select --install
```

---

## 下一步

安装完成后，您可以：

- [快速开始]({% link quickstart.md %}) - 学习基本使用
- [命令参考]({% link commands/index.md %}) - 查看所有可用命令
- [示例教程]({% link examples.md %}) - 实际应用场景

---

## 获取帮助

如果您在安装过程中遇到问题：

1. 查看 [故障排除]({% link troubleshooting.md %})
2. 在 [GitHub Issues](https://github.com/peeka-project/peeka/issues) 提问
