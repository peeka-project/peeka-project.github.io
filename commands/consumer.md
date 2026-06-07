---
layout: default
title: consumer 命令
parent: 命令参考
nav_order: 20
---

# consumer 命令
{: .no_toc }

管理结果消费者。consumer 为 job、probe 或 target 建立有界缓冲区，供 CLI、TUI、MCP 或 API 客户端读取诊断结果流。
{: .fs-6 .fw-300 }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 命令格式

```bash
peeka-cli consumer <subcommand> [options]
```

## 子命令

| 子命令 | 说明 |
|--------|------|
| `create` | 创建结果消费者 |
| `list` | 列出结果消费者 |
| `status --consumer <id>` | 查看消费者状态 |
| `drain --consumer <id>` | 读取缓冲记录 |
| `close --consumer <id>` | 关闭消费者 |
| `cleanup` | 清理 closed/failed 消费者 |

## create 参数

| 参数 | 说明 |
|------|------|
| `--target <id>` | 所属 target |
| `--source cli/tui/mcp/api/internal` | 请求来源 |
| `--scope-type job/probe/target` | 消费范围类型 |
| `--scope-id <id>` | job、probe 或 target ID |
| `--client <id>` | 可选所属客户端 |
| `--max-buffer-size <n>` | 最大缓冲记录数，默认 `1000` |
| `--backpressure-policy drop_oldest/drop_newest/fail` | 缓冲满时的处理策略，默认 `drop_oldest` |

## drain 参数

| 参数 | 说明 |
|------|------|
| `--limit <n>` | 最多读取记录数，默认 `100` |
| `--after-sequence <n>` | 只返回 sequence 大于该值的记录 |
| `--timeout-ms <n>` | 最多等待新记录的毫秒数，默认 `0` |

所有子命令支持 `--format table` 或 `--format json`。

## 示例

```bash
peeka-cli consumer create \
  --target target_abcd1234 \
  --source cli \
  --scope-type probe \
  --scope-id probe_123 \
  --format json

peeka-cli consumer drain --consumer consumer_123 --limit 50 --format json
peeka-cli consumer close --consumer consumer_123
peeka-cli consumer cleanup --target target_abcd1234
```

## 版本历史

| 版本 | 发布日期 | 变更说明 |
|------|----------|----------|
| 0.1.16 | 2026-06-07 | 新增 `consumer` 命令组 |

## 相关命令

- [client]({% link commands/client.md %}) - 管理客户端会话
- [job]({% link commands/job.md %}) - 管理命令任务
- [probe]({% link commands/probe.md %}) - 管理探针运行
