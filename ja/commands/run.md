---
layout: default
title: run コマンド
parent: コマンドリファレンス
nav_order: 14
---

# run コマンド
{: .no_toc }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}


## 概要

`run` コマンドは Python スクリプトを起動し、エントリーポイントの処理を実行する前に Peeka をアタッチして観測コマンドを設定します。短命なスクリプトやバッチ処理に適しています。

**実行順序（v0.1.22）**：スクリプトは元の順序で一度だけ実行されます。事前読み込みと後続の実行は同じモジュール名前空間を共有し、設定済みのプローブを保持します。トップレベルに `if __name__ == "__main__":` がある場合、その直前で停止してプローブを設定します。ない場合は、最後のトップレベル関数またはクラス定義まで実行してからプローブを設定し、残りの処理を続けます。

この境界より前のインポート、代入、その他の副作用は観測開始前に実行されます。初期化を観測するには、関数を先に定義し、事前読み込み部分ではなく main ガード内から呼び出してください。

## ユースケース

- **エントリーポイントの初期化**：初期化関数を main ガード内から呼び出し、プローブ設定後に実行する
- **短命なスクリプト**：プロセスが終了するまでの時間が短く、手動 attach が間に合わない場合
- **バッチジョブ**：cronタスク、データパイプライン、ワンショットスクリプトの診断
- **CI/CD デバッグ**：コードを変更せずに自動化パイプラインで関数の動作を捕捉
- **起動時のバグ再現**：起動条件を正確に制御して、起動時にのみ発生する問題を再現

## コマンド形式

```bash
peeka-cli run <script> [script_args] -- <command> [command_options]
```

`--` は必須のセパレーターです。左側はスクリプトとその引数、右側は Peeka コマンドとそのオプションです。

### パラメーター

| パラメーター       | 説明                                         | デフォルト |
|--------------------|----------------------------------------------|-----------|
| `script`           | 実行する Python スクリプトのパス              | —         |
| `script_args`      | スクリプトに渡す引数（任意）                  | —         |
| `--`               | 必須のセパレーター                            | —         |
| `command`          | Peeka コマンド（watch / trace / stack / monitor / top） | —         |
| `command_options`  | Peeka コマンドのオプション                    | —         |
| `--output-file`    | JSONL 出力をファイルに書き込む（stdout の代わり）| —      |

### サポートされるサブコマンド

| サブコマンド | 用途                                   |
|-------------|----------------------------------------|
| `watch`     | 関数呼び出しを観測（引数・戻り値・時間）|
| `trace`     | タイミング付きコールツリーを追跡        |
| `stack`     | 関数エントリ時にコールスタックを捕捉    |
| `monitor`   | 関数呼び出し指標を定期的に出力          |
| `top`       | 関数レベルのサンプリング profiler を開始 |

## 使用例

### 例 1: 最もシンプルな使い方

```bash
peeka-cli run myscript.py -- watch "mymodule.MyClass.init_db"
```

プローブ設定後に発生する `init_db` の呼び出しを観測します。

### 例 2: スクリプト引数を渡す

```bash
peeka-cli run myscript.py --env production --config /etc/app.yml -- watch "mymodule.func"
```

`--env` と `--config` はスクリプトへ、`watch "mymodule.func"` は Peeka コマンドです。

### 例 3: コールツリーのトレース

```bash
peeka-cli run myscript.py -- trace "mymodule.func"
```

`mymodule.func` の直接の子呼び出しと集計時間を追跡します。`-d/--depth` は使用できません。

### 例 4: 条件フィルター付き

```bash
peeka-cli run myscript.py -- watch "mymodule.func" --condition "params[0] > 100"
```

最初の引数が 100 より大きい呼び出しのみ捕捉します。

### 例 5: ファイルへ出力

```bash
peeka-cli run --output-file observations.jsonl myscript.py -- watch "mymodule.func"
```

全観測データを `observations.jsonl` に書き込みます。スクリプト自体の stdout には影響しません。

### 例 6: 観測回数の制限

```bash
peeka-cli run myscript.py -- watch "mymodule.func" -n 10
```

10 回捕捉後、自動的に観測を停止します。

### 例 7: コールスタックの捕捉

```bash
peeka-cli run myscript.py -- stack "mymodule.func" -n 3
```

`mymodule.func` のエントリ時にコールスタックを 3 回捕捉します。

### 例 8: monitor を開始

```bash
peeka-cli run myscript.py -- monitor "service.process" --interval 5 -c 12
```

5 秒ごとに関数統計を出力し、12 サイクル後に停止します。

### 例 9: top を開始

```bash
peeka-cli run myscript.py -- top -i 0.02 -c 10 --sort total
```

スクリプトを関数レベルでサンプリングし、10 個の snapshot を出力します。

## run と attach の違い

| 特徴                 | `run`                           | `attach`                        |
|----------------------|---------------------------------|---------------------------------|
| プロセス状態         | Peeka が起動                    | 実行中のプロセスに接続           |
| 観測範囲             | プローブ設定後の処理         | attach 時点から                  |
| 最適な用途           | 短命なスクリプト、初期化ロジック | 長期間稼動するサービス           |
| 起動方法             | `peeka-cli run script.py -- …`  | `peeka-cli attach <pid>`        |
| ptrace/PEP768 必要   | 必要（子プロセスへアタッチ）                | 必要                            |

## 注意事項

### ⚠️ `--` セパレーターは必須

```bash
# ✅ 正しい: -- でスクリプト引数と Peeka コマンドを分離
peeka-cli run myscript.py arg1 -- watch "mymodule.func"

# ❌ 誤り: -- がないと解析エラー
peeka-cli run myscript.py arg1 watch "mymodule.func"
```

### ⚠️ スクリプト終了時に観測も終了

スクリプトが終了（正常・異常いずれも）すると、観測は自動的に停止します。継続的な観測には、長期稼動プロセスへの `attach` を使用してください。

### --output-file は Peeka の出力のみに影響

`--output-file` は Peeka の JSONL 診断データのみを指定ファイルに書き込みます。スクリプト自身の stdout/stderr は影響を受けません。これは `run` レベルのオプションなので、スクリプトパスの前に置いてください。

## 関連コマンド

- [`attach`](attach.md) - 実行中のプロセスにアタッチ
- [`watch`](watch.md) - 関数呼び出しの観測
- [`trace`](trace.md) - コールツリーの追跡
- [`stack`](stack.md) - コールスタックの捕捉

## 更新履歴

| バージョン | 日付       | 更新内容           |
|-----------|------------|-------------------|
| 0.1.22 | 2026-11-17 | 元の文の順序で一度だけ実行し、共有名前空間でプローブを保持。事前読み込みと観測開始の境界を明記 |
| 0.1.18    | 2026-06-24 | `run` がすべてのストリーミング観測コマンド（watch/trace/stack/monitor/top）に対してストリーム単位のローカル制限を適用し、制限に達すると対応するストリームを停止 |
| 0.1.16    | 2026-06-07 | `run` が `monitor` と `top` をサポート。`--output-file` はスクリプトパス前の `run` レベルオプションとして修正 |
| 0.1.8     | 2025-04-28 | run コマンドのドキュメント追加 |
