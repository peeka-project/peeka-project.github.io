---
layout: default
title: profile コマンド
parent: コマンドリファレンス
nav_order: 22
---

# profile コマンド
{: .no_toc }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 概要

`profile` コマンドは、ターゲットプロセス内で **profiling セッション（ProfileSession）をオンデマンドで作成・確認・エクスポート** します。リアルタイムに更新される [`top`]({% link commands/top.md %}) と異なり、profile セッションはサンプルを agent 内に保持するため、いつでもスナップショットを取得したり、JSON、テキスト、[speedscope](https://www.speedscope.app/)、[Perfetto](https://ui.perfetto.dev/) 形式でエクスポートしてオフライン分析や共有に使えます。

`top` 自体も一時的な ProfileSession にサンプリングします。デフォルトでは `top` 終了時に削除されますが、`top --keep-profile` を使うと保持され、`profile` でエクスポートできます。

## 構文

```bash
peeka-cli attach <pid>    # 最初にターゲットプロセスへアタッチ
peeka-cli profile <subcommand> [options]
```

## サブコマンド

| サブコマンド | 説明 |
|--------------|------|
| `start` | profiling セッションを開始し、`profile_id` を返す |
| `stop --profile-id <id>` | セッションを停止し、収集済みデータを保持する |
| `snapshot --profile-id <id>` | 現在のデータのスナップショットを取得する |
| `list` | すべての profile セッションを一覧表示する |
| `cleanup` | 古いセッションを削除する |
| `export --profile-id <id>` | セッションデータをエクスポートする（ファイル出力も可） |

### start のオプション

| オプション | 説明 | デフォルト |
|------------|------|------------|
| `--backend sampling/exact` | profiling バックエンド | `sampling` |
| `--experimental` | 実験的な `exact` バックエンドを許可する | 無効 |
| `--interval <秒>` | サンプリング間隔 | `0.01` |
| `--duration <秒>` | 任意の実行時間。経過すると自動停止 | 無制限 |
| `--max-samples <N>` | 任意のサンプル上限。到達すると自動停止 | `100000` |

### snapshot / export のオプション

| オプション | 説明 | デフォルト |
|------------|------|------------|
| `--profile-id <id>` | セッション ID（必須） | - |
| `--format json/text/speedscope/perfetto` | 出力形式 | `json` |
| `--output <path>` | `export` のみ：エクスポート内容をこのファイルに書き出す | 標準出力 |

### cleanup のオプション

| オプション | 説明 |
|------------|------|
| `--older-than <秒>` | N 秒より前に作成されたセッションを削除する |
| `--max-sessions <N>` | 最大 N 個のセッションを保持する |

## バックエンド

- **sampling（デフォルト）**：バックグラウンドスレッドが `--interval` ごとに全スレッドのコールスタックをサンプリングします。オーバーヘッドが低く、本番プロセスでの長時間実行に向いています。
- **exact（実験的）**：Python の呼び出しイベントからすべての関数呼び出しを記録します。Python 3.12+（`threading.setprofile_all_threads`）が必要で、オーバーヘッドが明らかに高く、`--experimental` の指定が必須です。イベント数には上限があり、到達するとセッションは自動的に終了します。

`exact` と `sampling` は排他です。一方の実行中はもう一方を開始できないため、exact フックのオーバーヘッドがサンプリング結果を歪めることはありません。ターゲットプロセスに独自の profiler（`sys.setprofile` / `threading.setprofile`）がすでに設定されている場合、`exact` は置き換えずに開始を拒否します。

## 例

```bash
# 30 秒間サンプリング
peeka-cli profile start --duration 30

# セッション一覧
peeka-cli profile list

# speedscope 形式でエクスポートし、https://www.speedscope.app/ で開く
peeka-cli profile export --profile-id <profile_id> --format speedscope --output ./profile.speedscope.json

# 停止とクリーンアップ
peeka-cli profile stop --profile-id <profile_id>
peeka-cli profile cleanup --max-sessions 5
```

profile セッションを DX ケースに追加する：

```bash
peeka-cli dx add-profile --dx-case dx_123 --profile-id <profile_id>
```

## TUI での使用

TUI で `0` を押して **Performance** タブを開き、**Profiles** サブタブに切り替えると profile セッション一覧（`profile_id`、`backend`、`status`、`started_at`、`sample_count`）が表示されます。行を選択して **Export** をクリックするとエクスポートでき、`r` で更新します。詳しくは [TUI ガイド]({% link tui.md %}) を参照してください。

## 注意事項

- profile セッションはターゲットプロセスの agent 内に保持されます。最大 50 セッションまで保持され、超えた分は古いものから削除されます。`detach` はアクティブな profile をすべて停止します。
- `speedscope` 形式のエクスポートでは、スレッドごとに個別の profile が生成されます。
- 本番環境ではデフォルトの `sampling` バックエンドを推奨します。

## バージョン履歴

| バージョン | リリース日 | 変更内容 |
|------------|------------|----------|
| 0.1.21 | 2026-11-09 | `profile` コマンドグループ（start / stop / snapshot / list / cleanup / export）を追加。sampling と実験的な exact バックエンド、JSON / text / speedscope / Perfetto 形式に対応 |

## 関連コマンド

- [top]({% link commands/top.md %}) - リアルタイムの関数性能サンプリング
- [dx]({% link commands/dx.md %}) - 診断ケースバンドル
- [detach]({% link commands/detach.md %}) - 切断してリソースをクリーンアップ
