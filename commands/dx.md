---
layout: default
title: dx 命令
parent: 命令参考
nav_order: 21
---

# dx 命令
{: .no_toc }

创建、组织和导出诊断案例包。DX case 用于把 target、client、job、probe、consumer、错误、备注和摘要收集到同一个可导出的上下文中。
{: .fs-6 .fw-300 }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 命令格式

```bash
peeka-cli dx <subcommand> [options]
```

## 子命令

| 子命令 | 说明 |
|--------|------|
| `create --target <id> --title <text>` | 创建 DX case |
| `list` | 列出 DX case |
| `status --dx-case <id>` | 查看状态 |
| `add --dx-case <id>` | 添加一个 section |
| `summary --dx-case <id>` | 生成摘要 |
| `export --dx-case <id>` | 导出 DX case |
| `close --dx-case <id>` | 关闭 DX case |

常用参数：

| 参数 | 说明 |
|------|------|
| `--target <id>` | 所属 target；`create` 必填，其他子命令可选 |
| `--client <id>` | 可选所属客户端，用于访问控制 |
| `--format table/json` | 输出格式，默认 `table` |
| `--output-path <path>` | `export` 的可选输出路径 |

## 添加 section

```bash
peeka-cli dx add \
  --dx-case dx_123 \
  --section-type note \
  --title "Investigation note" \
  --payload-json '{"text":"slow query reproduced"}'
```

`--section-type` 支持 `target`、`client`、`job`、`probe`、`consumer`、`note`、`error` 和 `summary`。可以用 `--object-ref-type` 与 `--object-ref-id` 关联现有对象。

## 示例

```bash
peeka-cli dx create --target target_abcd1234 --title "Slow request"
peeka-cli dx list --target target_abcd1234
peeka-cli dx summary --dx-case dx_123
peeka-cli dx export --dx-case dx_123 --output-path ./slow-request.dx.json
peeka-cli dx close --dx-case dx_123
```

## 版本历史

| 版本 | 发布日期 | 变更说明 |
|------|----------|----------|
| 0.1.16 | 2026-06-07 | 新增 `dx` 命令组 |

## 相关命令

- [target]({% link commands/target.md %}) - 管理 target
- [job]({% link commands/job.md %}) - 收集命令任务信息
- [probe]({% link commands/probe.md %}) - 收集探针事件
