---
layout: default
title: dx コマンド
parent: コマンドリファレンス
nav_order: 21
---

# dx コマンド
{: .no_toc }

診断ケースバンドルを作成、整理、エクスポートします。DX case は target、client、job、probe、consumer、error、note、summary の section を 1 つのエクスポート可能なコンテキストに集約します。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 概要

`dx` コマンドは **診断ケースバンドル（DX case）の作成・整理・エクスポート** を行います：target、client、job、probe、consumer、エラー、メモ、サマリーを 1 つのエクスポート可能なコンテキストに集約し、事後分析とチーム連携を容易にします。

`create`、`add`、`summary`、`export` などのサブコマンドで、診断セッション中のあらゆる手がかりを単一の自己完結アーティファクトに集約でき、ターミナルやセッションをまたいで情報が散逸するのを防げます。エクスポート済みの DX case は issue や PR に再現エビデンスとして添付するのにも便利です。

## 構文

```bash
peeka-cli dx <subcommand> [options]
```

## サブコマンド

| サブコマンド | 説明 |
|--------------|------|
| `create --target <id> --title <text>` | DX case を作成 |
| `list` | DX case を一覧表示 |
| `status --dx-case <id>` | case 状態を表示 |
| `add --dx-case <id>` | section を追加 |
| `summary --dx-case <id>` | summary を生成 |
| `export --dx-case <id>` | DX case をエクスポート |
| `close --dx-case <id>` | DX case を閉じる |

共通オプション：

| オプション | 説明 |
|------------|------|
| `--target <id>` | 所有 target。`create` では必須、他のサブコマンドでは任意 |
| `--client <id>` | アクセス制御用の任意の所有クライアント |
| `--format table/json` | 出力形式。デフォルトは `table` |
| `--output-path <path>` | `export` の任意の出力先パス |

## section を追加

```bash
peeka-cli dx add \
  --dx-case dx_123 \
  --section-type note \
  --title "Investigation note" \
  --payload-json '{"text":"slow query reproduced"}'
```

`--section-type` は `target`、`client`、`job`、`probe`、`consumer`、`note`、`error`、`summary` を受け付けます。既存オブジェクトへリンクするには `--object-ref-type` と `--object-ref-id` を使います。

## 例

```bash
peeka-cli dx create --target target_abcd1234 --title "Slow request"
peeka-cli dx list --target target_abcd1234
peeka-cli dx summary --dx-case dx_123
peeka-cli dx export --dx-case dx_123 --output-path ./slow-request.dx.json
peeka-cli dx close --dx-case dx_123
```

## バージョン履歴

| バージョン | リリース日 | 変更 |
|------------|------------|------|
| 0.1.16 | 2026-06-07 | `dx` コマンドグループを追加 |

## 関連コマンド

- [target]({% link commands/target.md %}) - target を管理
- [job]({% link commands/job.md %}) - コマンドジョブ情報を収集
- [probe]({% link commands/probe.md %}) - probe イベントを収集
