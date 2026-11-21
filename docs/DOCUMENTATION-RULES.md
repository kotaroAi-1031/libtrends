# ドキュメント管理ルール（contextlint）

このリポジトリの調査資料（`docs/research/`）は、構造化 Markdown として
[contextlint](https://github.com/nozomi-koborinai/contextlint) で
整合性を検証する。ルールの正はリポジトリ直下の
`contextlint.config.json` で、このファイルはその運用ルールを人に
説明するためのものである。

## 実行方法

```bash
npx @contextlint/cli@1.1.1
```

対象は `docs/research/*.md`（著作文書のみ）。`docs/research/references/`
の repomix パックは他リポジトリのコードそのものなので対象外とする。

## 資料の構成契約

調査資料は次の6ファイルを必須とする（STR-001 で存在を検証する）。

| ファイル | 役割 |
| --- | --- |
| `docs/research/README.md` | 索引。資料一覧・結論サマリ・未決事項を持つ |
| `docs/research/01-apis.md` | 集計 API の網羅調査。API 一覧表が中心 |
| `docs/research/02-existing-implementations.md` | 既存実装（repomix 3点）の分析と比較 |
| `docs/research/03-health-signals.md` | uzomuzo / vuls を健全性指標に組み込む設計 |
| `docs/research/04-design-draft.md` | libtrends の設計たたき台 |
| `docs/research/05-icons.md` | アイコン取得元の調査と使い分け |

各ファイルの必須セクションは SEC-001 でファイルごとに検証する。
番号つき資料は末尾に「出典」セクションを持ち、事実には出典 URL を
付けることを原則とする。

## ID の規約

- `01-apis.md` の API 一覧表で、各 API に `API-001` 形式の ID を振る
  （列名は `ID`、パターンは `^API-\d{3}$`）。
- ID の重複は TBL-006 で禁止する。
- `04-design-draft.md` と `README.md` から API を参照するときは
  この ID で参照する。未定義 ID への参照と、どこからも参照されない
  定義は REF-002 が検出する。

## API 一覧表の規約

`01-apis.md` の「API一覧」セクションの表は、次の列を必須とする
（TBL-001）。空セルは置かない（TBL-002）。分からない項目は空にせず
「不明（要確認）」と書く。

料金の列は、無料／無料枠あり（有償プラン）／従量課金などの区分を公式情報に
基づいて書き、価格と条件は「個別の補足」に書く。公式に料金情報が確認できない
ものは「不明（要確認）」とし、推測で埋めない。

| 列 | 内容 |
| --- | --- |
| ID | `API-001` 形式の一意な ID |
| API | API の名称 |
| 提供指標 | 取れる指標（ダウンロード数、スター数など） |
| 認証 | 要否と方式 |
| レート制限 | 上限と単位 |
| 料金 | 無料・有料プラン・従量課金などの区分（公式情報に基づく） |
| 出典 | 公式ドキュメントの URL |

## その他のルール

- REF-001 / REF-005: 資料間のリンクとアンカーの実在を検証する。
- CTX-001: `TBD` などのプレースホルダの残置を検出する。
  書きかけを残す場合はプレースホルダではなく「未決事項」に書く。
- GRP-003: どこからもリンクされない孤立資料を検出する。
  新しい資料を作ったら索引（`docs/research/README.md`）から
  リンクすること。

## ルールの変更

ルールを変えるときは `contextlint.config.json` とこのファイルの
両方を同じプルリクエストで更新する。ルールを緩める変更は、
なぜ緩めるのかをプルリクエストの本文に書く。
