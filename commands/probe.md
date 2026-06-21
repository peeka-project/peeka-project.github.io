---
layout: default
title: probe 命令
parent: 命令参考
nav_order: 19
---

# probe 命令
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 简介

`probe` 命令用于**管理探针运行**。`watch`、`trace`、`monitor`、`top` 等观测命令在执行时会在 target 内注册 probe，`probe` 命令是这些 probe 的统一控制入口：查看注册情况、检查最近事件、停止运行中的 probe 并清理旧 probe。

通过 `list`、`status`、`inspect`、`stop`、`cleanup` 子命令可以按 target、类型（`watch`、`trace` 等）或状态过滤 probe，及时回收忘记关闭的观测，避免对 target 造成持续开销。

## 命令格式

```bash
peeka-cli probe <subcommand> [options]
```

## 子命令

| 子命令 | 说明 |
|--------|------|
| `list` | 列出 probe run |
| `status --probe <id>` | 查看 probe 状态 |
| `inspect --probe <id>` | 查看 probe 详情和最近事件 |
| `stop --probe <id>` | 停止运行中的 probe |
| `cleanup` | 清理旧 probe |

常用选项：

| 参数 | 说明 |
|------|------|
| `--target <id>` | 按拥有 probe 的 target 过滤或定位 |
| `--type <type>` | `list` 时按 probe 类型过滤，如 `watch`、`trace` |
| `--status <status>` | `list` 时按状态过滤 |
| `--events <n>` | `inspect` 返回最近事件数量，默认 `100` |
| `--all` | `cleanup` 时也清理 created/paused probe；不会清理 active probe |
| `--older-than <duration>` | `cleanup` 时清理早于指定时间的 probe，默认 `10m` |
| `--format table/json` | 输出格式，默认 `table` |

## 示例

```bash
peeka-cli probe list --target target_abcd1234
peeka-cli probe list --type watch --status active
peeka-cli probe inspect --probe probe_123 --events 20 --format json
peeka-cli probe stop --probe probe_123
peeka-cli probe cleanup --older-than 1h
```

## 版本历史

| 版本 | 发布日期 | 变更说明 |
|------|----------|----------|
| 0.1.16 | 2026-06-07 | 新增 `probe` 命令组 |

## 相关命令

- [watch]({% link commands/watch.md %}) - 创建函数观测
- [trace]({% link commands/trace.md %}) - 创建调用链追踪
- [job]({% link commands/job.md %}) - 管理命令任务
