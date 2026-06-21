---
layout: default
title: job 命令
parent: 命令参考
nav_order: 18
---

# job 命令
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 简介

`job` 命令用于**管理异步命令任务**：查看命令生命周期、检查结果元数据、中断运行中的任务并清理历史任务。`watch`、`trace`、`monitor`、`top` 等长生命周期观测命令通常在后台以 job 形式运行，`job` 命令是它们的统一控制入口。

通过 `list`、`status`、`inspect`、`interrupt`、`cleanup` 子命令可以在不直接接触 target 内部状态的前提下回收资源、定位卡顿任务，并按 target / client / 状态多维度过滤历史记录。

## 命令格式

```bash
peeka-cli job <subcommand> [options]
```

## 子命令

| 子命令 | 说明 |
|--------|------|
| `list` | 列出命令任务 |
| `status --job <id>` | 查看任务状态 |
| `inspect --job <id>` | 查看任务完整详情 |
| `interrupt --job <id>` | 中断运行中的任务 |
| `cleanup` | 清理历史任务 |
| `pull --job <id> --consumer <name>` | 预留的结果拉取接口；当前为 Phase 5 stub |

常用过滤和选项：

| 参数 | 说明 |
|------|------|
| `--target <id>` | 按拥有任务的 target 过滤或定位 |
| `--client <id>` | `list` 时按客户端过滤 |
| `--status <status>` | `list` 时按任务状态过滤 |
| `--completed` | `cleanup` 时只清理已完成任务 |
| `--older-than <duration>` | `cleanup` 时清理早于指定时间的任务，默认 `10m` |
| `--format table/json` | 输出格式，默认 `table` |

`--older-than` 支持秒数或带单位的持续时间，例如 `30s`、`10m`、`2h`。

## 示例

```bash
peeka-cli job list --target target_abcd1234
peeka-cli job status --job job_123 --format json
peeka-cli job inspect --job job_123 --target target_abcd1234
peeka-cli job interrupt --job job_123
peeka-cli job cleanup --completed --older-than 30m
```

## 版本历史

| 版本 | 发布日期 | 变更说明 |
|------|----------|----------|
| 0.1.16 | 2026-06-07 | 新增 `job` 命令组 |

## 相关命令

- [probe]({% link commands/probe.md %}) - 管理探针运行
- [consumer]({% link commands/consumer.md %}) - 读取缓冲结果
- [target]({% link commands/target.md %}) - 选择 target
