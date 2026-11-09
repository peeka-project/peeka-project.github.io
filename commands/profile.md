---
layout: default
title: profile 命令
parent: 命令参考
nav_order: 22
---

# profile 命令
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 简介

`profile` 命令用于在目标进程中**按需创建、查看和导出 profiling 会话**（ProfileSession）。与实时滚动的 [`top`]({% link commands/top.md %}) 不同，profile 会话会把采样数据保存在 agent 中，之后可以随时生成快照，或导出为 JSON、文本、[speedscope](https://www.speedscope.app/) 和 [Perfetto](https://ui.perfetto.dev/) 格式，便于离线分析和分享。

`top` 本身也基于临时的 ProfileSession 采样：默认在 `top` 结束时清理，使用 `top --keep-profile` 可以保留该会话，再用 `profile` 命令导出。

## 命令格式

```bash
peeka-cli attach <pid>    # 首先附加到目标进程
peeka-cli profile <subcommand> [options]
```

## 子命令

| 子命令 | 说明 |
|--------|------|
| `start` | 启动一个 profiling 会话，返回 `profile_id` |
| `stop --profile-id <id>` | 停止会话并保留已采集的数据 |
| `snapshot --profile-id <id>` | 生成当前数据的快照 |
| `list` | 列出所有 profile 会话 |
| `cleanup` | 清理旧的会话 |
| `export --profile-id <id>` | 导出会话数据，可写入文件 |

### start 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--backend sampling/exact` | profiling 后端 | `sampling` |
| `--experimental` | 允许使用实验性的 `exact` 后端 | 关闭 |
| `--interval <秒>` | 采样间隔 | `0.01` |
| `--duration <秒>` | 可选的持续时间，到期后自动停止 | 不限 |
| `--max-samples <N>` | 可选的最大样本数，达到后自动停止 | `100000` |

### snapshot / export 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--profile-id <id>` | 目标会话 ID（必填） | - |
| `--format json/text/speedscope/perfetto` | 输出格式 | `json` |
| `--output <path>` | 仅 `export`：把导出内容写入该文件 | 输出到标准输出 |

### cleanup 参数

| 参数 | 说明 |
|------|------|
| `--older-than <秒>` | 删除创建时间早于 N 秒的会话 |
| `--max-sessions <N>` | 最多保留 N 个会话 |

## 后端

- **sampling（默认）**：后台线程按 `--interval` 周期采样所有线程的调用栈，开销低，适合在生产进程上长时间运行。
- **exact（实验性）**：基于 Python 调用事件记录每次函数调用，需要 Python 3.12+（`threading.setprofile_all_threads`），开销明显更高，必须同时指定 `--experimental`。事件数量有上限，达到上限后会自动结束。

`exact` 与 `sampling` 互斥：其中一个运行时，另一个无法启动，避免 exact 钩子的开销干扰采样结果。如果目标进程已经安装了自己的 profiler（`sys.setprofile` / `threading.setprofile`），`exact` 会拒绝启动，而不是覆盖它。

## 示例

```bash
# 采样 30 秒
peeka-cli profile start --duration 30

# 查看会话
peeka-cli profile list

# 导出为 speedscope 格式，在 https://www.speedscope.app/ 中打开
peeka-cli profile export --profile-id <profile_id> --format speedscope --output ./profile.speedscope.json

# 停止并清理
peeka-cli profile stop --profile-id <profile_id>
peeka-cli profile cleanup --max-sessions 5
```

把 profile 会话附加到诊断案例包：

```bash
peeka-cli dx add-profile --dx-case dx_123 --profile-id <profile_id>
```

## TUI 使用

在 TUI 中按 `0` 打开 **Performance** 标签页，切换到 **Profiles** 子标签即可查看 profile 会话列表（`profile_id`、`backend`、`status`、`started_at`、`sample_count`），选中一行后可点击 **Export** 导出，按 `r` 刷新。详见 [TUI 使用指南]({% link tui.md %})。

## 注意事项

- profile 会话保存在目标进程的 agent 中，最多保留 50 个会话，超出时会淘汰最旧的会话；`detach` 会停止所有活动的 profile。
- 导出 `speedscope` 格式时，每个线程会生成独立的 profile。
- 生产环境优先使用默认的 `sampling` 后端。

## 版本历史

| 版本 | 发布日期 | 变更说明 |
|------|----------|----------|
| 0.1.21 | 2026-11-09 | 新增 `profile` 命令组（start / stop / snapshot / list / cleanup / export），支持 sampling 与实验性 exact 后端，以及 JSON / text / speedscope / Perfetto 格式 |

## 相关命令

- [top]({% link commands/top.md %}) - 实时函数性能采样
- [dx]({% link commands/dx.md %}) - 诊断案例包
- [detach]({% link commands/detach.md %}) - 断开连接并清理资源
