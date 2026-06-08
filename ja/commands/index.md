---
layout: default
title: コマンドリファレンス
nav_order: 4
has_children: true
permalink: /commands
---

# コマンドリファレンス
{: .no_toc }

Peeka は一連の強力な診断コマンドを提供し、それぞれが特定の診断シナリオに特化しています。このドキュメントでは 21 個のコマンドリファレンストピックを扱います。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## コマンド概要

| コマンド | 機能 | 使用場面 |
|---------|----------|----------|
| attach | ターゲットプロセスにアタッチ | すべてのシナリオの最初のステップ |
| watch | 関数呼び出しをオブザーブ | パラメータ、戻り値、実行時間を確認 |
| trace | 呼び出しチェーンをトレース | 関数呼び出しの関係と時間分布を分析 |
| stack | 呼び出しスタックをトレース | 関数が誰に呼び出されたかを追跡 |
| monitor | パフォーマンス統計 | 関数パフォーマンス指標をリアルタイムモニタリング |
| logger | ログ管理 | 実行時にログレベルを動的調整 |
| memory | メモリ分析 | メモリ使用量とメモリリークを分析 |
| inspect | オブジェクト検査 | 実行時のオブジェクトプロパティを検査 |
| search (sc/sm) | クラスとメソッドを検索 | コード探索と発見 |
| reset | 拡張をリセット | オブザーブされた関数を復元 |
| thread | スレッド分析 | スレッドを列挙してスタックを表示 |
| top | 関数プロファイリング | 関数レベルのパフォーマンスホットスポット分析 |
| patch-status | ランタイムパッチ診断 | gevent/eventlet、stdlib プリミティブ、RPL 整合性を確認 |
| detach | 切断 | 診断セッションを安全に終了 |
| run | 起動してアタッチ | Python プログラムを起動し、自動的に診断セッションに入る |
| target | ターゲット管理 | Peeka target agent の検出、検査、クリーンアップ、デタッチ |
| client | クライアントセッション管理 | 診断クライアントの作成、一覧、検査、終了 |
| job | コマンドジョブ管理 | 非同期コマンドジョブの一覧、検査、中断、クリーンアップ |
| probe | プローブ実行管理 | watch/trace 系プローブの一覧、検査、停止、クリーンアップ |
| consumer | 結果コンシューマ管理 | job/probe/target のバッファ済み結果ストリームを作成・読み取り |
| dx | 診断ケースバンドル | 診断ケースの作成、要約、エクスポート |

`session` は `target` の互換エイリアスとして残っていますが非推奨です。新しいワークフローでは `peeka-cli target ...` を使用してください。

---

## 共通パラメータ

すべてのコマンドは以下のパラメータフォーマットを共有しています：

### パターンフォーマット

ターゲット関数パターンを指定するために使用されます：

```bash
# クラスメソッド
module.ClassName.method_name

# モジュール関数
module.function_name

# ワイルドカードサポート
module.ClassName.*
module.*
*.method_name
```

### 出力フォーマット

すべてのコマンドは JSONL（JSON Lines）フォーマットを出力し、1行ごとに 1つの JSON オブジェクトです：

```json
{"type":"status","level":"info","message":"..."}
{"type":"success","command":"attach","data":{...}}
{"type":"observation","watch_id":"...","data":{...}}
```

### メッセージタイプ

| タイプ | 説明 |
|------|-------------|
| `status` | ステータス情報（重要でない） |
| `success` | コマンド成功 |
| `error` | コマンド失敗 |
| `event` | 制御イベント（開始、停止） |
| `observation` | オブザベーションデータ |
| `result` | クエリ結果 |

---

## コマンド使用フロー

### 標準診断フロー

```bash
# 1. プロセスにアタッチ
peeka-cli attach <pid>

# 2. 特定の診断コマンドを使用
peeka-cli watch "module.func"

# 3. 結果を分析
peeka-cli watch "module.func" | jq 'select(.type == "observation")'

# 4. (オプション) 拡張をリセット
peeka-cli reset "module.func"
```

### TUI インタラクティブフロー

```bash
# TUI を起動
peeka

# 数字キーでビューを切り替え
# 1 - Dashboard
# 2 - Watch ビュー
# 3 - Trace ビュー
# 4 - Stack ビュー
# 5 - Monitor ビュー
# 6 - Memory ビュー
# 7 - Logger ビュー
# 8 - Inspect ビュー
# 9 - Threads ビュー
# 0 - Top ビュー
```

詳細は [TUI 使用ガイド]({% link tui.md %}) を参照してください。

---

## 条件式

多くのコマンドは `--condition` パラメータをサポートして、オブザベーション結果をフィルタリングできます。

### 使用可能な変数

| 変数 | 説明 | 型 |
|----------|-------------|------|
| `params` | 関数パラメータリスト | list |
| `kwargs` | キーワード引数辞書 | dict |
| `returnObj` | 戻り値 | any |
| `throwExp` | 例外オブジェクト | Exception or None |
| `cost` | 実行時間（ミリ秒） | float |
| `target` | ターゲットオブジェクト（インスタンスメソッド） | object |

### 条件式の例

```bash
# パラメータフィルタリング
--condition "params[0] > 100"
--condition "len(params) > 2"
--condition "kwargs.get('debug') == True"

# 戻り値フィルタリング
--condition "returnObj is not None"
--condition "len(returnObj) > 10"

# 実行時間フィルタリング
--condition "cost > 100"  # 100ms 以上
--condition "cost > 10 and cost < 100"  # 10-100ms

# 例外フィルタリング
--condition "throwExp is not None"
--condition "type(throwExp).__name__ == 'ValueError'"

# 複合条件
--condition "params[0] > 100 and cost > 50"
--condition "len(params) > 2 or returnObj is None"
```

### セキュリティ制限

条件式は `simpleeval` ライブラリを使用して安全に評価され、以下はサポートされていません：

- ❌ `__import__`、`eval`、`exec` などの危険な関数
- ❌ ファイル操作（`open`、`read`、`write`）
- ❌ リフレクション操作（`__class__`、`__subclasses__`）
- ✅ 算術、比較、論理演算
- ✅ 文字列操作、リストインデックス
- ✅ 安全なビルトイン関数（`len`、`str`、`int` など）

---

## パフォーマンスへの影響

| コマンド | パフォーマンスオーバーヘッド | 注意 |
|---------|---------------------|-------|
| `watch` | < 1% | デコレータ注入、最小オーバーヘッド |
| `trace` | < 5% (3.12+) | `sys.monitoring` API を使用 |
| `trace` | < 20% (3.8.1-3.11) | `sys.settrace` を使用 |
| `stack` | < 1% | 呼び出しスタックだけをキャプチャ |
| `monitor` | < 1% | 定期的な統計 |
| `logger` | 0% | パフォーマンス影響なし |
| `memory` | 設定可能 | サンプリング頻度に依存 |
| `patch-status` | ほぼ 0% | ランタイム状態を読むだけで、対象プロセスは変更しない |
| `target` / `client` / `job` / `probe` / `consumer` / `dx` | 低 | 制御プレーンの問い合わせまたはメタデータ操作。読み取りやエクスポートするデータ量に依存 |

---

## 次のステップ

必要なコマンドを選択して詳細なドキュメントを確認してください：

- [attach - ターゲットプロセスにアタッチ]({% link commands/attach.md %})
- [watch - 関数呼び出しをオブザーブ]({% link commands/watch.md %})
- [trace - 呼び出しチェーンをトレース]({% link commands/trace.md %})
- [stack - 呼び出しスタックをトレース]({% link commands/stack.md %})
- [monitor - パフォーマンスモニタリング]({% link commands/monitor.md %})
- [logger - ログ管理]({% link commands/logger.md %})
- [memory - メモリ分析]({% link commands/memory.md %})
- [inspect - オブジェクト検査]({% link commands/inspect.md %})
- [search - クラスとメソッドを検索]({% link commands/search.md %})
- [reset - 拡張をリセット]({% link commands/reset.md %})
- [thread - スレッド分析]({% link commands/thread.md %})
- [top - 関数プロファイリング]({% link commands/top.md %})
- [patch-status - ランタイムパッチ診断]({% link commands/patch-status.md %})
- [detach - 切断]({% link commands/detach.md %})
- [run - 起動してアタッチ]({% link commands/run.md %})
- [target - ターゲット管理]({% link commands/target.md %})
- [client - クライアントセッション管理]({% link commands/client.md %})
- [job - コマンドジョブ管理]({% link commands/job.md %})
- [probe - プローブ実行管理]({% link commands/probe.md %})
- [consumer - 結果コンシューマ管理]({% link commands/consumer.md %})
- [dx - 診断ケースバンドル]({% link commands/dx.md %})

クイックスタートは [クイックスタート]({% link quickstart.md %}) で基本的な使用方法を学んでください。
