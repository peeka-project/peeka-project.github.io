---
layout: default
title: patch-status 命令
parent: 命令参考
nav_order: 15
---

# patch-status 命令
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 简介

`patch-status` 是只读诊断命令，用于确认目标进程是否被 gevent/eventlet monkey patch、当前 socket/thread/time 等 stdlib 原语是否仍与 Runtime Primitive Layer（RPL）捕获的原生实现一致，以及当前 asyncio loop 和线程模型状态。

该命令不会修复或撤销 monkey patch。它只报告状态，便于判断 attach、watch、trace 等诊断命令在复杂运行时环境中的行为。

## 命令格式

```bash
peeka-cli attach <pid>
peeka-cli patch-status [--pid <pid>]
```

| 参数 | 说明 |
|------|------|
| `--pid` | 可选兼容参数；当前实现会忽略该值，并报告当前已附加会话的状态 |

必须先完成 `attach`。如果当前没有活动会话，命令会返回错误并提示先运行 `peeka-cli attach <pid>`。

## 输出字段

成功时，`patch-status` 返回 `result` 消息，内部数据包含以下字段：

| 字段 | 说明 |
|------|------|
| `schema_version` | 输出 schema 版本，当前为 `"1"` |
| `pid` | 目标进程 PID |
| `timestamp` | 采样时间戳 |
| `monkey_patch` | gevent/eventlet 是否已导入、是否处于 active 状态，以及已 patch 模块列表 |
| `stdlib_origin` | 当前 stdlib 原语与 RPL 捕获的原生原语 ID 对比 |
| `asyncio_loop` | 当前 asyncio loop 是否运行、policy 和 loop class |
| `thread_model` | 主线程、线程总数、daemon 线程数和线程模型分类 |
| `rpl_integrity` | RPL 捕获原语是否完整，以及是否发生 drift |

## 示例

```bash
peeka-cli attach 12345
peeka-cli patch-status | jq '.data.data'
```

示例输出：

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

## 典型场景

- 在 gevent 或 eventlet 服务中 attach 前后确认 monkey patch 状态
- 排查 socket、threading、time 等原语被替换导致的 attach 或观测异常
- 验证 RPL 是否仍保有原生 socket/thread/time 原语
- 确认目标进程当前是否运行 asyncio loop，以及线程模型是否符合预期

## 解读建议

- `monkey_patch.gevent.status == "active"` 或 `monkey_patch.eventlet.status == "active"` 表示目标进程启用了对应 monkey patch。
- `stdlib_origin.*.matches == false` 表示当前 stdlib 对象已不同于 RPL 捕获的原生对象；这通常说明运行时被 patch 过。
- `rpl_integrity.ok == true` 表示 RPL 自身捕获的原生原语仍可用于 Peeka 内部诊断路径。
- `patch-status` 不会恢复原生函数；如果需要清理 Peeka 注入的观测增强，请使用 [reset]({% link commands/reset.md %}) 或 [detach]({% link commands/detach.md %})。

## 版本历史

| 版本 | 发布日期 | 变更说明 |
|------|----------|----------|
| 0.1.14 | 2026-05-24 | 新增 `patch-status` 运行时诊断命令 |

## 相关命令

- [attach]({% link commands/attach.md %}) - 附加到目标进程
- [watch]({% link commands/watch.md %}) - 观测函数调用
- [trace]({% link commands/trace.md %}) - 追踪调用链
- [reset]({% link commands/reset.md %}) - 重置增强
