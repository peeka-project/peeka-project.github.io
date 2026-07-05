---
layout: default
title: trace コマンド
parent: コマンドリファレンス
nav_order: 3
permalink: /commands/trace
---

# trace コマンド
{: .no_toc }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}


## 概要

`trace` コマンドは、Python 関数の**直接の呼び出し先**と実行時間を追跡し、ターゲット関数の次の 1 階層のサブ呼び出しをフラットな集計リストとして表示します。性能ボトルネックの特定やコード実行ホットスポットの理解に役立つ強力な性能分析ツールです。

**v0.1.20 の重要な変更**：`trace` は深さ指定オプションをサポートしなくなり、再帰的にネストした呼び出しツリーも出力しません。バックエンドはターゲット関数の直接の呼び出し先（direct callees）のみを捕捉・集計するため、出力がより安定し、オーバーヘッドも予測しやすくなりました。


## TUI での使用法

TUI モードでは **`3`** キーを押して **Trace ビュー**に切り替えると、以下のインタラクティブ機能が利用できます：

- **パターン入力**：ターゲットプロセスからリアルタイムに取得した関数名のオートコンプリートをサポート
- **パラメータ設定**：最小実行時間、回数、条件式、skip-builtin を視覚的に設定可能
- **アクティブ追跡リスト**：上部に現在の trace タスクの状態と件数を表示
- **呼び出しツリーの可視化**：各観測の直接の呼び出し先をインタラクティブなツリーで表示し、集計された callee ノードを含みます
- **詳細統計パネル**：選択中の観測または callee の実行時間、回数、割合、例外情報を表示
- **実行時間による色分け**：
  - 🟢 緑：< 10ms（高速）
  - 🟡 黄：10-100ms（中速）
  - 🔴 赤：>= 100ms（低速）
- **ショートカット操作**：
  - 入力モード後に Enter でトレース開始
  - アクティブ追跡リストで行を選択し、その pattern の呼び出しツリーを表示
  - 呼び出しツリーで callee ノードを選択し、`t` キーでその関数を下位追跡（Drill Trace）
  - `c` キーでトレース記録をクリアし、実行中の trace を停止
  - Delete キーですべての trace を停止

![Peeka Trace ビュー]({{ site.url }}/assets/images/screenshots/peeka-trace.png)

**CLI との同等性**：以下の例はすべて CLI コマンドでデモンストレーションしていますが、TUI は同じ機能をグラフィカルなインターフェースで提供します。

## 使用場面

- **性能ボトルネックの特定**：実行時間データからターゲット関数配下で最も遅いサブ呼び出しを素早く発見
- **ホットスポット関数の識別**：直接の呼び出し先について合計時間、呼び出し回数、最小/最大時間を集計
- **コード実行パスの追跡**：さまざまな条件下でのコード実行パスを観察
- **サブ関数の時間分析**：各サブ関数の時間消費割合を分析
- **例外呼び出しの診断**：例外を投げた呼び出しを追跡し、例外タイプを確認

## コマンドフォーマット

```bash
peeka-cli attach <pid>    # まずターゲットプロセスにアタッチ
peeka-cli trace <pattern> [options]
```

### パラメータ説明

| パラメータ                    | 説明                    | デフォルト     | 例                                      |
|-----------------------|-----------------------|---------|-----------------------------------------|
| `pattern`             | 関数マッチングパターン                | -       | `module.Class.method`                   |
| `-n, --times`         | 観測回数（-1 は無限）         | `-1`    | `-n 10`                                 |
| `--condition` | 条件式（`cost` 変数をサポート）   | なし       | `--condition "cost > 50"`       |
| `--client`            | 既存のクライアントセッション ID。省略時は一時クライアントを自動作成 | 自動 | `--client client_123`                  |
| `--skip-builtin`      | 組み込み関数と標準ライブラリ関数をスキップ          | `true`  | `--skip-builtin=false`                  |
| `--min-duration`      | 最小実行時間フィルタ（ミリ秒）。この値以上の直接呼び出しのみを記録 | `0`     | `--min-duration 10`                     |

**注意**：
- `--skip-builtin` はデフォルトで有効、出力のノイズを減らすため
- 条件式の `cost` 変数は呼び出し全体の合計時間（ミリ秒）を表します
- v0.1.20 以降、深さ指定オプションはサポートされません

### 関数マッチングパターン (pattern)

以下のフォーマットをサポートしています：

```python
# 1. モジュールレベルの関数
"mymodule.my_function"

# 2. クラスのメソッド
"mymodule.MyClass.my_method"

# 3. ネストされたクラスのメソッド
"mypackage.mymodule.OuterClass.InnerClass.method"

# 4. モジュールパス
"package.subpackage.module.function"
```

**注意**：インポートのルートから始まる完全なモジュールパスを使用する必要があり、現在のバージョンではワイルドカードマッチングはサポートされていません。

## 基本的な使い方

### 1. 関数呼び出しのトレース

```bash
# まずターゲットプロセスにアタッチ
peeka-cli attach 12345

# 5 回の呼び出しをトレース
peeka-cli trace "calculator.Calculator.calculate" -n 5
```

**出力例**：

```json
{
  "type": "observation",
  "watch_id": "trace_abc123",
  "timestamp": 1705586200.123,
  "func_name": "calculator.Calculator.calculate",
  "location": "AtExit",
  "call_tree": [
    {
      "function": "calculator.Calculator._validate",
      "filename": "/app/calculator.py",
      "lineno": 18,
      "count": 1,
      "total_ms": 2.1,
      "min_ms": 2.1,
      "max_ms": 2.1
    },
    {
      "function": "calculator.Calculator._compute",
      "filename": "/app/calculator.py",
      "lineno": 25,
      "count": 1,
      "total_ms": 98.2,
      "min_ms": 98.2,
      "max_ms": 98.2
    },
    {
      "function": "calculator.Logger.info",
      "filename": "/app/logger.py",
      "lineno": 10,
      "count": 1,
      "total_ms": 15.7,
      "min_ms": 15.7,
      "max_ms": 15.7
    }
  ],
  "total_duration_ms": 125.3,
  "self_time_ms": 9.3,
  "callee_count": 3,
  "node_count": 4,
  "thread_id": 140234567890,
  "thread_name": "MainThread"
}
```

**フィールド説明**：

| フィールド                 | 説明           | 例値                       |
|--------------------|--------------|---------------------------|
| `watch_id`         | 観測 ID        | `"trace_abc123"`          |
| `timestamp`        | タイムスタンプ          | `1705586200.123`          |
| `func_name`        | ターゲット関数名        | `"calculator.Calculator.calculate"`  |
| `location`         | 観測位置         | `"AtExit"`                |
| `call_tree`        | 直接の呼び出し先リスト（フラット集計）    | `[...]`                   |
| `total_duration_ms`| 合計実行時間（ミリ秒）    | `125.3`                   |
| `self_time_ms`     | ターゲット関数自身の実行時間（ミリ秒） | `9.3`                     |
| `callee_count`     | 直接の呼び出し先の種類数      | `3`                       |
| `node_count`       | ノード総数（ターゲット関数 + 直接の呼び出し先） | `4`                       |
| `thread_id`        | スレッド ID         | `140234567890`            |
| `thread_name`      | スレッド名          | `"MainThread"`            |
| `exception`        | 例外情報（発生時）    | `"ValueError: ..."`       |
| `runtime_meta`     | ランタイムメタデータ（バックエンド、gevent など） | `{...}`        |

**呼び出しツリーノードのフィールド**：

| フィールド            | 説明         | 例値                    |
|---------------|------------|------------------------|
| `function`    | 関数の完全名     | `"module.Class.method"`|
| `filename`    | ファイルパス       | `"/app/module.py"`     |
| `lineno`      | 行番号         | `42`                   |
| `count`       | この観測期間内の呼び出し回数 | `5`                    |
| `total_ms`    | 合計実行時間（ミリ秒）   | `125.3`                |
| `min_ms`      | 最小実行時間（ミリ秒）    | `10.5`                 |
| `max_ms`      | 最大実行時間（ミリ秒）    | `95.1`                 |

### 2. 視覚的な呼び出しツリー（TUI）

TUI モードでは、Trace ビューは上下レイアウトを採用します：

![Peeka Trace 呼び出しツリー]({{ site.url }}/assets/images/screenshots/peeka-trace.png)

**説明**：
- 上部の Active Traces には現在の追跡タスク（Pattern / Status / Count）が表示されます
- 下部左側の Call Tree には、選択中 pattern の観測ノード、集計 callee ノード、各 callee の実行時間割合が表示されます
- 下部右側の Stats パネルには、現在選択中の観測または callee の詳細統計が表示されます
- 色分けによって実行時間の範囲が強調されます
- callee を選択して `t` キーを押すと、新しい下位追跡を素早く開始できます

### 3. 最小実行時間でフィルタリング

```bash
# 実行時間 >= 10ms の直接呼び出しだけを記録
peeka-cli trace "service.process" --min-duration 10
```

これにより、高頻度で短時間の補助関数によるノイズを減らし、実際にリソースを消費している可能性のあるサブ呼び出しに集中できます。

### 4. 条件フィルタリング

```bash
# 合計時間が 50ms を超える呼び出しだけをトレース
peeka-cli trace "api.handler" --condition "cost > 50"

# パラメータと時間の条件を組み合わせ
peeka-cli trace "service.query" --condition "cost > 100 and params[0] > 1000"
```

### 5. 組み込み関数のスキップ

```bash
# デフォルトの動作：組み込み関数をスキップ（出力のノイズを減らす）
peeka-cli trace "mymodule.func"

# すべての呼び出しを表示（組み込み関数も含む）
peeka-cli trace "mymodule.func" --skip-builtin=false
```

**組み込み関数の例**：
- Python 組み込み関数：`len()`, `str()`, `isinstance()`, `print()`
- 標準ライブラリ関数：`json.dumps()`, `os.path.join()`, `datetime.now()`

## 実装技術

### 実装原理

Peeka の `trace` コマンドは、Python バージョンに応じて自動的に最適な実装方法を選択します：

| Python バージョン | 実装方案            | 性能オーバーヘッド  | 説明                |
|-----------|-----------------|-------|-------------------|
| 3.12+     | sys.monitoring  | < 5%  | 公式 PEP 669 API、最適な性能         |
| 3.8.1-3.11  | sys.settrace | < 20% | 互換性が良く、自動的に有効化         |

**gevent 互換性（v0.1.15+）**：target プロセスで gevent monkey patch または active hub が検出された場合、`trace` は `sys.settrace` が frame stack 不変条件を壊すことを避けるため `wrapper_only` バックエンドに退化します。このモードでも target 関数の観測結果は報告されますが、再帰的な呼び出しツリーは提供されません。

**直接の呼び出し先セマンティクス（v0.1.20+）**：すべてのバックエンドはターゲット関数の直接の呼び出し先のみを捕捉し、同じ観測期間内の同一 `(function, filename, lineno)` 呼び出しを集計して、`count / total_ms / min_ms / max_ms` を出力します。これにより、再帰的・深層の呼び出しツリーによる性能の不確実性とデータ膨張を避けられます。

**sys.monitoring 実装 (Python 3.12+)**:
- [PEP 669](https://peps.python.org/pep-0669/) に基づく公式モニタリング API
- `PY_START` と `PY_RETURN` イベントを使用して呼び出しをキャプチャ
- 性能オーバーヘッド < 5% で、本番環境での使用が推奨
- 複数の観測が自動的にツール ID を割り当てられ、競合しない

**sys.settrace 実装 (Python 3.8.1-3.11)**:
- Python 組み込みの `sys.settrace()` メカニズムを使用
- ターゲット関数の実行中にのみ有効（部分トレース）
- 性能オーバーヘッド < 20% で、ほとんどのシナリオで十分に利用可能

**skip-builtin フィルタリングメカニズム**:
- `code.co_filename.startswith('<')` をチェックして組み込み関数（`<built-in>`）をフィルタリング
- Python 標準ライブラリパスをチェックし、標準ライブラリ関数をフィルタリング
- デフォルトで有効、50% 以上の出力ノードを削減できる

## 性能への影響

### 性能オーバーヘッド

| シナリオ                  | オーバーヘッド    | 説明            |
|---------------------|-------|---------------|
| **単純な関数**            | < 5%  | Python 3.12+  |
| **単純な関数**            | < 20% | Python 3.8.1-3.11 |
| **高頻度のサブ呼び出し**    | 10-30%| Python バージョンと `--min-duration` 設定に応じて  |
| **高頻度呼び出し（>1000 QPS）** | 20-50%| 推奨されるのは観測回数の制限      |

**説明**：
- Python 3.12+ では `sys.monitoring` を使用するため、性能オーバーヘッドが大幅に削減される
- v0.1.20 以降は 1 階層の直接呼び出し先のみを追跡するため、オーバーヘッドはより安定し予測しやすい
- 本番環境では条件フィルタリングと回数制限を使用することが推奨される

### 性能最適化の提案

1. **最小実行時間フィルタを使用する**
   ```bash
   # 実行時間 >= 10ms の直接呼び出しだけを記録
   peeka-cli trace "func" --min-duration 10
   ```

2. **組み込み関数をスキップする**
   ```bash
   # デフォルトで有効、50% 以上のノードを削減
   peeka-cli trace "func" --skip-builtin
   ```

3. **条件フィルタリングを使用する**
   ```bash
   # 遅い呼び出しだけをトレース
   peeka-cli trace "func" --condition "cost > 100"
   ```

4. **観測回数を制限する**
   ```bash
   # 10 回だけ観測
   peeka-cli trace "func" -n 10
   ```

## 使用例

### 1. 性能ボトルネックの特定

```bash
# 遅いインターフェースをトレースし、最も時間のかかるサブ呼び出しを見つける
peeka-cli trace "api.handler.process_request" --condition "cost > 100"
```

**出力**：
```
[1250ms] api.handler.process_request()
├── [10ms] api.validator.check_params() (count=1)
├── [1200ms] database.query.execute()  ← ボトルネックはここ！ (count=1)
└── [20ms] api.formatter.to_json() (count=1)
```

**結論**：データベースクエリが 96% の時間を消費しているので、SQL の最適化やインデックスの追加が必要です。

### 2. 高頻度呼び出しの集計

```bash
# ループ内の関数を追跡し、同じサブ呼び出しが複数回発生する状況を確認
peeka-cli trace "algorithm.process_batch" -n 5
```

**出力例**：
```json
{
  "func_name": "algorithm.process_batch",
  "call_tree": [
    {
      "function": "database.query.fetch",
      "count": 100,
      "total_ms": 850.5,
      "min_ms": 5.1,
      "max_ms": 25.3
    }
  ]
}
```

**結論**：`process_batch` は 1 回の実行でデータベースクエリを 100 回発行しています。バッチクエリ最適化を検討してください。

### 3. コード実行パスの理解

```bash
# 条件分岐の実行パスをトレース
peeka-cli trace "service.business_logic" -n 1
```

**正常フローの場合**：
```
`---[50ms] service.business_logic()
    +---[5ms] service.validate_input()
    +---[30ms] service.process_data()
    `---[10ms] service.save_result()
```

**異常フローの場合**：
```
`---[20ms] service.business_logic()
    +---[5ms] service.validate_input()
    +---[10ms] service.handle_invalid_input()
    `---[3ms] service.log_error()
```

### 4. 最適化前後の性能比較

```bash
# 最適化前
peeka-cli trace "converter.parse_json" -n 10 > before.jsonl

# 最適化後
peeka-cli trace "converter.parse_json" -n 10 > after.jsonl

# 平均実行時間を分析
jq '.total_duration_ms' before.jsonl | awk '{sum+=$1; count++} END {print "Before:", sum/count, "ms"}'
jq '.total_duration_ms' after.jsonl | awk '{sum+=$1; count++} END {print "After:", sum/count, "ms"}'
```

### 5. CI/CD への統合

```bash
#!/bin/bash
# 性能回帰テスト
THRESHOLD=100  # 最大許容時間 100ms

peeka-cli attach $PID
RESULT=$(peeka-cli trace "critical.function" -n 50 | \
  jq -s 'map(select(.type == "observation")) | map(.total_duration_ms) | add / length')

if (( $(echo "$RESULT > $THRESHOLD" | bc -l) )); then
  echo "Performance regression detected: ${RESULT}ms > ${THRESHOLD}ms"
  exit 1
fi
```

## データ処理と分析

### jq を使用した処理

```bash
# 1. 呼び出しツリーを抽出
peeka-cli trace "func" | jq '.call_tree'

# 2. 平均実行時間を計算
peeka-cli trace "func" -n 100 | jq '.total_duration_ms' | \
  awk '{sum+=$1; count++} END {print "avg:", sum/count, "ms"}'

# 3. 最も遅いサブ呼び出しを見つける
peeka-cli trace "func" | jq '.call_tree | sort_by(.total_ms) | reverse | .[0]'

# 4. 呼び出し頻度を統計（count フィールドで集計）
peeka-cli trace "func" -n 100 | jq -s '[.[] | .call_tree[] | {function, count}] | group_by(.function) | map({function: .[0].function, total_count: map(.count) | add}) | sort_by(.total_count) | reverse'

# 5. フレームグラフ用データを生成（total_ms を使用）
peeka-cli trace "func" -n 1000 | jq -r '.call_tree[] | "\(.function) \(.total_ms)"' > flamegraph.txt
```

### Python による分析

```python
import json
import sys
from collections import defaultdict

# 直接の呼び出し先の合計時間と回数を統計
stats = defaultdict(lambda: {"count": 0, "total_ms": 0})

for line in sys.stdin:
    data = json.loads(line)
    if data["type"] == "observation":
        for callee in data.get("call_tree", []):
            func = callee.get("function")
            if func:
                stats[func]["count"] += callee.get("count", 1)
                stats[func]["total_ms"] += callee.get("total_ms", 0)

# 合計時間でソート
sorted_stats = sorted(stats.items(), key=lambda x: x[1]["total_ms"], reverse=True)

print("Top 10 Time-Consuming Functions:")
print(f"{'Function':<60} {'Count':>10} {'Total (ms)':>15} {'Avg (ms)':>12}")
print("-" * 100)

for func, stat in sorted_stats[:10]:
    avg_ms = stat["total_ms"] / stat["count"] if stat["count"] else 0
    print(f"{func:<60} {stat['count']:>10} {stat['total_ms']:>15.2f} {avg_ms:>12.2f}")
```

**実行**：
```bash
peeka-cli trace "module.func" -n 100 | python analyze_trace.py
```

**出力**：
```
Top 10 Time-Consuming Functions:
Function                                                      Count     Total (ms)      Avg (ms)
----------------------------------------------------------------------------------------------------
database.query.execute                                          100       12500.00       125.00
api.handler.process_request                                     100       15000.00       150.00
json.dumps                                                      500        1000.00         2.00
...
```

## よくある問題

### 1. なぜ深い階層の呼び出しが表示されないのですか？

**問題**：呼び出しツリーにはターゲット関数の直接の呼び出し先だけが表示され、サブ呼び出しのさらに下の呼び出しが表示されません。

**原因**：v0.1.20 以降、`trace` は直接の呼び出し先（direct-callee セマンティクス）のみを捕捉・集計します。これは、より安定した性能と予測しやすい出力を提供するためです。

**解決策**：

```bash
# サブ呼び出し内部を観察したい場合は、そのサブ呼び出しに対して個別に trace を開始
peeka-cli trace "module.sub_module.slow_func" -n 10
```

### 2. 出力データが多すぎる

**問題**：組み込み関数の呼び出しが大量に含まれ、出力が読みにくい。

**解決策**：

```bash
# 組み込み関数をスキップ（デフォルトで有効）
peeka-cli trace "module.func" --skip-builtin

# 10ms を超える呼び出しだけを記録
peeka-cli trace "module.func" --min-duration 10

# 条件フィルタリングを使用
peeka-cli trace "module.func" --condition "cost > 50"
```

### 3. 性能オーバーヘッドが大きすぎる

**問題**：trace を有効にするとアプリケーションの応答が遅くなる。

**解決策**：

```bash
# 1. 最小実行時間しきい値を上げ、記録するノードを減らす
peeka-cli trace "module.func" --min-duration 10

# 2. 観測回数を制限する
peeka-cli trace "module.func" -n 10

# 3. 条件フィルタリングを使用し、遅い呼び出しだけをトレース
peeka-cli trace "module.func" --condition "cost > 100"

# 4. Python 3.12+ にアップグレードすると性能が向上します
```

### 4. データが観測できない

**考えられる原因**：
- 関数が呼び出されていない
- 関数名のスペルが間違っている
- 条件式が厳しすぎる
- 既に観測回数制限（-n パラメータ）に達している

**調査手順**：

```bash
# 1. 関数名が正しいか確認
python3 -c "import mymodule; print(mymodule.MyClass.my_method)"

# 2. 条件式を削除して、まず 1 回観測
peeka-cli trace "mymodule.func" -n 1

# 3. プロセスが存在するか確認
ps aux | grep <pid>
```

## 高度なテクニック

### 1. フレームグラフの生成

```bash
# トレースデータを収集
peeka-cli trace "module.func" -n 1000 > trace.jsonl

# フレームグラフ形式に変換（直接の呼び出し先を total_ms で折りたたむ）
jq -r '.call_tree[] | "\(.function);\(.total_ms)"' trace.jsonl \
  > folded.txt

# フレームグラフを生成（flamegraph.pl が必要）
flamegraph.pl folded.txt > flamegraph.svg
```

### 2. 複数バージョンの性能比較

```bash
# バージョン A
git checkout v1.0
peeka-cli trace "module.func" -n 100 > trace_v1.jsonl

# バージョン B
git checkout v2.0
peeka-cli trace "module.func" -n 100 > trace_v2.jsonl

# 平均実行時間を比較
echo "v1.0: $(jq -s 'map(.total_duration_ms) | add / length' trace_v1.jsonl) ms"
echo "v2.0: $(jq -s 'map(.total_duration_ms) | add / length' trace_v2.jsonl) ms"
```

### 3. 自動化性能モニタリング

```python
#!/usr/bin/env python3
"""性能回帰監視スクリプト"""
import json
import subprocess
import time

THRESHOLD = 100  # 最大許容時間 (ミリ秒)
CHECK_INTERVAL = 3600  # チェック間隔 (秒)

def check_performance(pid, pattern):
    cmd = ["peeka-cli", "trace", pattern, "-n", "50"]
    proc = subprocess.Popen(cmd, stdout=subprocess.PIPE, text=True)

    durations = []
    for line in proc.stdout:
        data = json.loads(line)
        if data["type"] == "observation":
            durations.append(data["total_duration_ms"])

    avg_duration = sum(durations) / len(durations) if durations else 0

    if avg_duration > THRESHOLD:
        send_alert(f"Performance regression: {avg_duration:.2f}ms > {THRESHOLD}ms")

    return avg_duration

def send_alert(message):
    # アラートを送信（メール、Slack、钉钉など）
    print(f"ALERT: {message}")

if __name__ == "__main__":
    pid = int(sys.argv[1])
    pattern = sys.argv[2]

    while True:
        duration = check_performance(pid, pattern)
        print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] Avg duration: {duration:.2f}ms")
        time.sleep(CHECK_INTERVAL)
```

### 4. Prometheus への統合

```python
from prometheus_client import Histogram, start_http_server
import json
import subprocess

# メトリクスを定義
trace_duration = Histogram('trace_duration_ms', 'Function trace duration', ['function'])

# Prometheus サーバーを起動
start_http_server(8000)

# トレースデータを収集
proc = subprocess.Popen(
    ["peeka-cli", "trace", "module.func"],
    stdout=subprocess.PIPE,
    text=True
)

for line in proc.stdout:
    data = json.loads(line)
    if data["type"] == "observation":
        # 直接の呼び出し先リストを処理
        for callee in data.get("call_tree", []):
            func = callee.get("function")
            total_ms = callee.get("total_ms", 0)
            if func:
                trace_duration.labels(function=func).observe(total_ms)
```

## 参考資料

- [PEP 669: Low Impact Monitoring for CPython](https://peps.python.org/pep-0669/)
- [Peeka アーキテクチャ設計](../architecture.md)

## 更新履歴

| バージョン    | 日付      | 更新内容          |
|-------|---------|---------------|
| 0.2.0 | 2026-02 | trace コマンドのドキュメントを追加 |
| 0.1.0 | 2025-01 | 初期バージョン          |

## 更新履歴

| バージョン | リリース日 | 変更内容 |
|----------|----------|---------|
| 0.1.20 | 2026-07-05 | 深さ指定オプションを削除。trace バックエンドはターゲット関数の直接の呼び出し先（direct callees）のみを捕捉・集計し、`call_tree` をフラットリストとして出力。`self_time_ms`、`callee_count` フィールドを追加。TUI Trace ビューは上部 Active Traces リスト + 下部の呼び出しツリー/統計分割に変更し、callee 下位追跡ショートカット `t` と集計 callee ノードを追加 |
| 0.1.18 | 2026-06-24 | CLI `--times` がアクティブな stream_id に一致する観測のみをカウントし、並行するストリームの観測が現在の trace の制限に誤って含まれるのを防止。`run` ラッパーは制限に達すると trace ストリームを停止 |
| 0.1.17 | 2026-06-13 | trace レスポンスが `wrapper_only` バックエンドに降格された際、`runtime_meta`（`startup_backend`、`effective_backend`、`downgrade_reason`）を含むように。TUI の Trace ビューは統計パネルに Backend / Gevent 状態を表示 |
| 0.1.16 | 2026-06-07 | `--client` を追加 |
| 0.1.15 | 2026-05-27 | gevent patched/active hub ランタイムでは `wrapper_only` trace バックエンドに退化 |
| 0.1.12 | 2026-05-08 | TUI パネルシステムの統一、レスポンシブレイアウトの改善（commit 50c4af4） |
| 0.1.11 | 2026-05-07 | 安定ソースによるクライアントラベリング（commit 965ff22）、アクティビティ診断の強化（commit b1b0412） |
| 0.1.10 | 2026-05-04 | TUI ボタン色の正規化（commit fd6a0a1）、アクティビティログ折り返しの可読性向上（commit 5f46ae8） |
