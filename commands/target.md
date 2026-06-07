---
layout: default
title: target 命令
parent: 命令参考
nav_order: 16
---

# target 命令
{: .no_toc }

管理 Peeka target agent：发现当前主机上的 Peeka 目标、查看状态、清理 stale marker，并按 target ID 分离目标。
{: .fs-6 .fw-300 }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 命令格式

```bash
peeka-cli target <subcommand> [options]
```

`target` 面向已经被 Peeka 注入或管理的目标进程。目标状态包括 `alive`、`stale`、`unknown`、`attaching`、`failed` 和 `detached`。

`session` 是兼容旧脚本的弃用别名：

```bash
peeka-cli session list
```

新脚本请改用 `peeka-cli target ...`。

## 子命令

| 子命令 | 说明 |
|--------|------|
| `list` | 列出发现的 target agent |
| `current` | 当且仅当存在一个 `alive` target 时返回当前 target |
| `status --target <id>` | 查看指定 target 的摘要状态 |
| `inspect --target <id>` | 查看指定 target 的完整详情、能力和下一步动作 |
| `cleanup [--target <id>] [--dry-run]` | 清理 stale target marker；默认只清理 stale target |
| `detach --target <id> [--force]` | 分离指定 target；alive target 需要 `--force` |

所有子命令都支持：

| 参数 | 说明 |
|------|------|
| `--format table` | 默认表格输出 |
| `--format json` | 输出 JSONL 事件，便于脚本处理 |

## 示例

```bash
# 列出所有 target
peeka-cli target list

# 机器上只有一个 alive target 时返回它
peeka-cli target current

# 查看详情
peeka-cli target inspect --target target_abcd1234 --format json

# 预览 stale marker 清理
peeka-cli target cleanup --dry-run

# 分离 alive target
peeka-cli target detach --target target_abcd1234 --force
```

## current 退出码

| 退出码 | 含义 |
|--------|------|
| `0` | 恰好存在一个 alive target |
| `1` | 没有 alive target |
| `2` | 存在多个 alive target，需要显式选择 |

## 版本历史

| 版本 | 发布日期 | 变更说明 |
|------|----------|----------|
| 0.1.16 | 2026-06-07 | 新增 `target` 命令组；`session` 作为弃用兼容别名保留 |

## 相关命令

- [attach]({% link commands/attach.md %}) - 附加到目标进程
- [detach]({% link commands/detach.md %}) - 分离当前会话
- [client]({% link commands/client.md %}) - 管理客户端会话
