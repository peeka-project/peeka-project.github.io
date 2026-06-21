---
layout: default
title: target コマンド
parent: コマンドリファレンス
nav_order: 16
---

# target コマンド
{: .no_toc }

Peeka target agent を管理します。現在のホスト上の target を検出し、状態を確認し、stale marker を削除し、target ID でデタッチできます。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 概要

`target` コマンドは **Peeka の target agent を管理** します：現在のホスト上の Peeka target を発見し、状態を確認し、stale マーカーをクリーンアップし、target ID で対象を切り離します。他のすべての診断コマンドへの入り口でもあり、target が `alive` であると確認できて初めて、`watch`、`trace`、`inspect` などのコマンドを正しい対象プロセスへルーティングできます。

`list`、`current`、`status`、`inspect`、`cleanup`、`detach` サブコマンドを使うことで、同じホスト上に複数の target が共存する場面でも操作対象を正確に指定でき、プロセス終了後に socket マーカーだけが残った stale target を速やかに回収できます。

## 構文

```bash
peeka-cli target <subcommand> [options]
```

`target` は Peeka に注入または管理されているプロセスを対象にします。target の状態には `alive`、`stale`、`unknown`、`attaching`、`failed`、`detached` があります。

`session` は互換性のための非推奨エイリアスとして残っています：

```bash
peeka-cli session list
```

新しいスクリプトでは `peeka-cli target ...` を使用してください。

## サブコマンド

| サブコマンド | 説明 |
|--------------|------|
| `list` | 検出された target agent を一覧表示 |
| `current` | `alive` target が 1 つだけある場合に現在の target を返す |
| `status --target <id>` | target の状態概要を表示 |
| `inspect --target <id>` | target の詳細、capabilities、next actions を表示 |
| `cleanup [--target <id>] [--dry-run]` | stale target marker を削除。stale のみの削除がデフォルト |
| `detach --target <id> [--force]` | target をデタッチ。alive target には `--force` が必要 |

すべてのサブコマンドは以下をサポートします：

| オプション | 説明 |
|------------|------|
| `--format table` | デフォルトの表形式出力 |
| `--format json` | スクリプト向け JSONL イベント |

## 例

```bash
# すべての target を一覧表示
peeka-cli target list

# alive target が 1 つだけならそれを返す
peeka-cli target current

# 詳細を確認
peeka-cli target inspect --target target_abcd1234 --format json

# stale marker の削除をプレビュー
peeka-cli target cleanup --dry-run

# alive target をデタッチ
peeka-cli target detach --target target_abcd1234 --force
```

## current の終了コード

| 終了コード | 意味 |
|------------|------|
| `0` | alive target が 1 つだけ存在 |
| `1` | alive target が存在しない |
| `2` | alive target が複数存在。明示的な選択が必要 |

## バージョン履歴

| バージョン | リリース日 | 変更 |
|------------|------------|------|
| 0.1.16 | 2026-06-07 | `target` コマンドグループを追加。`session` は非推奨の互換エイリアスとして維持 |

## 関連コマンド

- [attach]({% link commands/attach.md %}) - target プロセスにアタッチ
- [detach]({% link commands/detach.md %}) - 現在のセッションをデタッチ
- [client]({% link commands/client.md %}) - クライアントセッションを管理
