---
layout: default
title: consumer コマンド
parent: コマンドリファレンス
nav_order: 20
---

# consumer コマンド
{: .no_toc }

結果コンシューマを管理します。consumer は job、probe、target の出力に有界バッファを作成し、CLI、TUI、MCP、API クライアントが診断結果ストリームを読み取れるようにします。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 構文

```bash
peeka-cli consumer <subcommand> [options]
```

## サブコマンド

| サブコマンド | 説明 |
|--------------|------|
| `create` | 結果コンシューマを作成 |
| `list` | 結果コンシューマを一覧表示 |
| `status --consumer <id>` | コンシューマ状態を表示 |
| `drain --consumer <id>` | バッファ済みレコードを読み取る |
| `close --consumer <id>` | コンシューマを閉じる |
| `cleanup` | closed または failed コンシューマを削除 |

## create オプション

| オプション | 説明 |
|------------|------|
| `--target <id>` | 所有 target |
| `--source cli/tui/mcp/api/internal` | リクエスト元 |
| `--scope-type job/probe/target` | 読み取るスコープ種別 |
| `--scope-id <id>` | job、probe、target ID |
| `--client <id>` | 任意の所有クライアント |
| `--max-buffer-size <n>` | 最大バッファレコード数。デフォルトは `1000` |
| `--backpressure-policy drop_oldest/drop_newest/fail` | バッファ満杯時のポリシー。デフォルトは `drop_oldest` |

## drain オプション

| オプション | 説明 |
|------------|------|
| `--limit <n>` | 返す最大レコード数。デフォルトは `100` |
| `--after-sequence <n>` | この sequence より大きいレコードだけ返す |
| `--timeout-ms <n>` | 新しいレコードを待つ最大ミリ秒。デフォルトは `0` |

すべてのサブコマンドは `--format table` または `--format json` をサポートします。

## 例

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

## バージョン履歴

| バージョン | リリース日 | 変更 |
|------------|------------|------|
| 0.1.16 | 2026-06-07 | `consumer` コマンドグループを追加 |

## 関連コマンド

- [client]({% link commands/client.md %}) - クライアントセッションを管理
- [job]({% link commands/job.md %}) - コマンドジョブを管理
- [probe]({% link commands/probe.md %}) - probe 実行を管理
