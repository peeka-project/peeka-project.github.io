---
layout: default
title: client コマンド
parent: コマンドリファレンス
nav_order: 17
---

# client コマンド
{: .no_toc }

target agent に紐づくクライアントセッションを作成・管理します。クライアントセッションは CLI、TUI、MCP、API、内部呼び出しを区別するために使います。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 概要

`client` コマンドは **target agent に紐づくクライアントセッションを作成・管理** し、CLI、TUI、MCP、API、内部呼び出しを区別します。各セッションには独立した ID があり、[job]({% link commands/job.md %}) や [consumer]({% link commands/consumer.md %}) の結果をクライアント単位で分離できます。

`--source` で呼び出し元をマークし、`list`、`status`、`close` でセッションのライフサイクルを追跡することで、複数クライアントが同じ target に対して並行して診断を行う場合でもコンテキストを明確に保てます。

## 構文

```bash
peeka-cli client <subcommand> [options]
```

## サブコマンド

| サブコマンド | 説明 |
|--------------|------|
| `create --target <id> --source <source>` | クライアントセッションを作成 |
| `list [--target <id>]` | クライアントセッションを一覧表示。target で絞り込み可能 |
| `status --client <id>` | クライアントセッションの状態を表示 |
| `close --client <id>` | クライアントセッションを閉じる |

`--source` には `cli`、`tui`、`mcp`、`api`、`internal` を指定できます。`create` は任意のユーザー識別子として `--user <id>` も受け付けます。

すべてのサブコマンドは `--format table`（デフォルト）または `--format json` をサポートします。

## 例

```bash
# CLI 自動化用のクライアントを作成
peeka-cli client create --target target_abcd1234 --source cli --format json

# 1 つの target のクライアントを一覧表示
peeka-cli client list --target target_abcd1234

# クライアントを確認して閉じる
peeka-cli client status --client client_123
peeka-cli client close --client client_123
```

## 典型的な用途

- 複数のツールが同じ target に接続するとき、呼び出し元ごとに独立した identity を持たせる。
- job、probe、consumer、dx 操作を特定のクライアントに関連付ける。
- 自動化で明示的な所有者やアクセス制御が必要な場合、先にクライアントを作成して `--client` を渡す。

## バージョン履歴

| バージョン | リリース日 | 変更 |
|------------|------------|------|
| 0.1.16 | 2026-06-07 | `client` コマンドグループを追加 |

## 関連コマンド

- [target]({% link commands/target.md %}) - target を管理
- [job]({% link commands/job.md %}) - コマンドジョブを管理
- [consumer]({% link commands/consumer.md %}) - 結果コンシューマを管理
