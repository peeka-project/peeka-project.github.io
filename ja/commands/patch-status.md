---
layout: default
title: patch-status コマンド
parent: コマンドリファレンス
nav_order: 15
---

# patch-status コマンド
{: .no_toc }

対象プロセスの monkey patch、stdlib プリミティブの由来、asyncio loop、スレッドモデル、RPL 整合性を検査します。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 概要

`patch-status` は読み取り専用の診断コマンドです。対象プロセスが gevent/eventlet で monkey patch されているか、socket/thread/time などの stdlib プリミティブが Runtime Primitive Layer（RPL）の捕捉したネイティブ実装と一致しているか、また asyncio loop とスレッドモデルがどの状態かを確認します。

このコマンドは monkey patch を修正または解除しません。複雑なランタイム環境で attach、watch、trace などの診断動作を判断するための状態を報告します。

## 構文

```bash
peeka-cli attach <pid>
peeka-cli patch-status [--pid <pid>]
```

| パラメータ | 説明 |
|------------|------|
| `--pid` | 互換用の任意パラメータ。現在の実装では無視され、アクティブなアタッチ済みセッションを報告します |

先に `attach` が必要です。アクティブなセッションがない場合、`peeka-cli attach <pid>` を実行するよう促すエラーを返します。

## 出力フィールド

成功時、`patch-status` は `result` メッセージを返し、内部データには以下が含まれます：

| フィールド | 説明 |
|------------|------|
| `schema_version` | 出力 schema バージョン。現在は `"1"` |
| `pid` | 対象プロセス PID |
| `timestamp` | サンプル時刻 |
| `monkey_patch` | gevent/eventlet の import 状態、active 状態、利用可能な場合は patch 済みモジュール一覧 |
| `stdlib_origin` | 現在の stdlib プリミティブ ID と RPL が捕捉したネイティブ ID の比較 |
| `asyncio_loop` | loop 実行状態、policy、loop class |
| `thread_model` | メインスレッド、総スレッド数、daemon スレッド数、分類 |
| `rpl_integrity` | RPL が捕捉したプリミティブが intact か、drift があるか |

## 例

```bash
peeka-cli attach 12345
peeka-cli patch-status | jq '.data.data'
```

出力例：

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

## 主な用途

- gevent または eventlet サービスに attach する前後で monkey patch 状態を確認する
- 置き換えられた socket、threading、time プリミティブによる attach やオブザーブの問題を調査する
- Peeka 内部で使うネイティブ socket/thread/time プリミティブが RPL に残っているか確認する
- 対象プロセスが asyncio loop を実行中か、スレッドモデルが想定どおりか確認する

## 結果の読み方

- `monkey_patch.gevent.status == "active"` または `monkey_patch.eventlet.status == "active"` は monkey patch が有効であることを示します。
- `stdlib_origin.*.matches == false` は、現在の stdlib オブジェクトが RPL の捕捉したネイティブオブジェクトと異なることを示します。
- `rpl_integrity.ok == true` は、RPL の捕捉したネイティブプリミティブが Peeka の内部診断経路で利用可能であることを示します。
- `patch-status` はネイティブ関数を復元しません。Peeka のオブザーブ拡張を削除するには [reset]({% link commands/reset.md %}) または [detach]({% link commands/detach.md %}) を使用してください。

## バージョン履歴

| バージョン | リリース日 | 変更内容 |
|-----------|-----------|----------|
| 0.1.14 | 2026-05-24 | `patch-status` ランタイム診断コマンドを追加 |

## 関連コマンド

- [attach]({% link commands/attach.md %}) - 対象プロセスにアタッチ
- [watch]({% link commands/watch.md %}) - 関数呼び出しをオブザーブ
- [trace]({% link commands/trace.md %}) - 呼び出しチェーンをトレース
- [reset]({% link commands/reset.md %}) - 拡張をリセット
