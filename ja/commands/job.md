---
layout: default
title: job コマンド
parent: コマンドリファレンス
nav_order: 18
---

# job コマンド
{: .no_toc }

非同期コマンドジョブを管理します。ライフサイクル状態の確認、結果メタデータの検査、実行中ジョブの中断、古いジョブのクリーンアップができます。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 構文

```bash
peeka-cli job <subcommand> [options]
```

## サブコマンド

| サブコマンド | 説明 |
|--------------|------|
| `list` | コマンドジョブを一覧表示 |
| `status --job <id>` | ジョブ状態を表示 |
| `inspect --job <id>` | ジョブの詳細を表示 |
| `interrupt --job <id>` | 実行中のジョブを中断 |
| `cleanup` | 古いジョブを削除 |
| `pull --job <id> --consumer <name>` | 予約済みの結果取得インターフェイス。現在は Phase 5 stub |

共通のフィルタとオプション：

| オプション | 説明 |
|------------|------|
| `--target <id>` | 所有 target で絞り込み、または特定 |
| `--client <id>` | `list` をクライアントセッションで絞り込み |
| `--status <status>` | `list` をジョブ状態で絞り込み |
| `--completed` | `cleanup` 時に完了済みジョブのみ削除 |
| `--older-than <duration>` | `cleanup` 時に指定期間より古いジョブを削除。デフォルトは `10m` |
| `--format table/json` | 出力形式。デフォルトは `table` |

`--older-than` は秒数または `30s`、`10m`、`2h` のような duration を受け付けます。

## 例

```bash
peeka-cli job list --target target_abcd1234
peeka-cli job status --job job_123 --format json
peeka-cli job inspect --job job_123 --target target_abcd1234
peeka-cli job interrupt --job job_123
peeka-cli job cleanup --completed --older-than 30m
```

## バージョン履歴

| バージョン | リリース日 | 変更 |
|------------|------------|------|
| 0.1.16 | 2026-06-07 | `job` コマンドグループを追加 |

## 関連コマンド

- [probe]({% link commands/probe.md %}) - probe 実行を管理
- [consumer]({% link commands/consumer.md %}) - バッファ済み結果を読み取る
- [target]({% link commands/target.md %}) - target を選択
