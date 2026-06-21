---
layout: default
title: client 命令
parent: 命令参考
nav_order: 17
---

# client 命令
{: .no_toc }

创建和管理绑定到 target agent 的客户端会话。客户端会话用于区分 CLI、TUI、MCP、API 或内部调用方。
{: .fs-6 .fw-300 }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 简介

`client` 命令用于**创建和管理绑定到 target agent 的客户端会话**，把 CLI、TUI、MCP、API 或内部调用方区分开。每个会话都有独立的 ID，便于 [job]({% link commands/job.md %}) 与 [consumer]({% link commands/consumer.md %}) 按客户端归属隔离结果。

通过 `--source` 标记调用来源，再配合 `list`、`status`、`close` 子命令查看会话生命周期，可在多客户端并发诊断的场景下保持上下文清晰。

## 命令格式

```bash
peeka-cli client <subcommand> [options]
```

## 子命令

| 子命令 | 说明 |
|--------|------|
| `create --target <id> --source <source>` | 创建客户端会话 |
| `list [--target <id>]` | 列出客户端会话，可按 target 过滤 |
| `status --client <id>` | 查看客户端会话状态 |
| `close --client <id>` | 关闭客户端会话 |

`--source` 可取 `cli`、`tui`、`mcp`、`api` 或 `internal`。`create` 还支持 `--user <id>` 记录可选用户标识。

所有子命令支持 `--format table`（默认）或 `--format json`。

## 示例

```bash
# 为 CLI 自动化创建客户端
peeka-cli client create --target target_abcd1234 --source cli --format json

# 查看某个 target 的客户端
peeka-cli client list --target target_abcd1234

# 查看和关闭客户端
peeka-cli client status --client client_123
peeka-cli client close --client client_123
```

## 典型用途

- 多个工具同时连接同一个 target 时，为每个调用方分配独立身份。
- 将 job、probe、consumer 或 dx 操作关联到具体客户端。
- 自动化脚本需要显式访问控制或结果归属时，先创建客户端再传递 `--client`。

## 版本历史

| 版本 | 发布日期 | 变更说明 |
|------|----------|----------|
| 0.1.16 | 2026-06-07 | 新增 `client` 命令组 |

## 相关命令

- [target]({% link commands/target.md %}) - 管理目标
- [job]({% link commands/job.md %}) - 管理命令任务
- [consumer]({% link commands/consumer.md %}) - 管理结果消费者
