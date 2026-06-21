---
layout: default
title: probe コマンド
parent: コマンドリファレンス
nav_order: 19
---

# probe コマンド
{: .no_toc }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 概要

`probe` コマンドは **プローブ実行を管理** します。`watch`、`trace`、`monitor`、`top` などの観測コマンドは実行時に target 内に probe を登録し、`probe` コマンドはそれら probe の統一コントロール窓口として機能します：登録状況の確認、直近イベントの調査、実行中 probe の停止、古い probe のクリーンアップが行えます。

`list`、`status`、`inspect`、`stop`、`cleanup` サブコマンドを使うと、target、タイプ（`watch`、`trace` など）、ステータスで probe をフィルタでき、閉じ忘れた観測をタイムリーに回収して target への継続的なオーバーヘッドを避けられます。

## 構文

```bash
peeka-cli probe <subcommand> [options]
```

## サブコマンド

| サブコマンド | 説明 |
|--------------|------|
| `list` | probe 実行を一覧表示 |
| `status --probe <id>` | probe 状態を表示 |
| `inspect --probe <id>` | probe の詳細と最近のイベントを表示 |
| `stop --probe <id>` | 実行中の probe を停止 |
| `cleanup` | 古い probe を削除 |

共通オプション：

| オプション | 説明 |
|------------|------|
| `--target <id>` | 所有 target で絞り込み、または特定 |
| `--type <type>` | `list` を `watch` や `trace` などの probe 種別で絞り込み |
| `--status <status>` | `list` を状態で絞り込み |
| `--events <n>` | `inspect` が返す最近のイベント数。デフォルトは `100` |
| `--all` | `cleanup` 時に created/paused probe も削除。active probe は削除しない |
| `--older-than <duration>` | `cleanup` 時に指定期間より古い probe を削除。デフォルトは `10m` |
| `--format table/json` | 出力形式。デフォルトは `table` |

## 例

```bash
peeka-cli probe list --target target_abcd1234
peeka-cli probe list --type watch --status active
peeka-cli probe inspect --probe probe_123 --events 20 --format json
peeka-cli probe stop --probe probe_123
peeka-cli probe cleanup --older-than 1h
```

## バージョン履歴

| バージョン | リリース日 | 変更 |
|------------|------------|------|
| 0.1.16 | 2026-06-07 | `probe` コマンドグループを追加 |

## 関連コマンド

- [watch]({% link commands/watch.md %}) - 関数観測を作成
- [trace]({% link commands/trace.md %}) - 呼び出しチェーントレースを作成
- [job]({% link commands/job.md %}) - コマンドジョブを管理
