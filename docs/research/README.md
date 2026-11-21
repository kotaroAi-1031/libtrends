# libtrends 調査資料

言語・レジストリ横断でライブラリの指標（ダウンロード数・スター・人気度・依存関係・保守性・
脆弱性）を集積するサイト「libtrends」の調査フェーズ成果物。2026-11 時点の一次情報
（公式ドキュメント）と実機検証にもとづく。

- 調査日: 2026-11-19（対象言語の絞り込みと料金調査は 2026-11-20 に追加）
- 対象言語: Go / TypeScript / JavaScript / MoonBit / Rust / Python の 6 言語。
  レジストリは npm（TS/JS）、PyPI、crates.io、Go module proxy、mooncakes.io（MoonBit）の 5 つ。
- 方針: 事実には出典 URL を付け、確認できない事項は「不明（要確認）」と明記する。
  推測・設計案はその旨を書く。
- 範囲: 本ディレクトリのみ。`references/` は読み取り専用の参考資料（repomix パック）。

## 資料一覧

| 資料 | 内容 |
| --- | --- |
| [01-apis.md](01-apis.md) | ライブラリ集計 API 17 件の網羅調査。提供指標・認証・レート制限・料金・出典と、選定方針（対象外 4 件の理由も記載） |
| [02-existing-implementations.md](02-existing-implementations.md) | npm.bet / TanStack NPM Stats / npm-trend-clone の分析と比較。設計の工夫と弱点 |
| [03-health-signals.md](03-health-signals.md) | uzomuzo（EOL・放棄検出）と vuls（脆弱性）を健全性指標に組み込む設計 |
| [04-design-draft.md](04-design-draft.md) | libtrends の設計たたき台（指標スキーマ・収集アーキテクチャ・画面構成） |
| [05-icons.md](05-icons.md) | アイコン取得元の調査。GitHub のアバターと OG 画像、パッケージアイコンのフォールバック、Simple Icons / Devicon / Iconify の重なりと使い分け |
| [summary.html](summary.html) | 01〜04 の結論を 1 ページにまとめた HTML（図解・コード例つき） |

参考資料（`references/`、他リポジトリのコードを repomix で固めたもの）:

- `npm-bet.md` — https://github.com/vercel/npm.bet
- `tanstack-npm-stats.md` — https://github.com/TanStack/tanstack.com
- `npm-trend-clone.md` — https://github.com/laststance/npm-trend-clone

## 結論サマリ

### 1. API は「一次ソース／横断集約／付加指標」の三層で使う

単一の API で全指標は揃わない。層を分けて組み合わせるのが現実的。

- **一次ソース（レジストリ公式）**: API-001（npm downloads）、API-002（npm registry）、
  API-009（PyPI JSON）、API-012（crates.io）、API-016（Go module proxy）、
  API-020（mooncakes.io / MoonBit）。更新が最も早いが、形式がバラバラで、
  ダウンロード数を返さないもの（Go）・累計のみのもの（MoonBit）もある。
- **横断集約**: API-004（deps.dev）をパッケージ同定と依存関係の基準に据える。認証不要で
  npm / PyPI / Go / Cargo を同一スキーマで扱える（**MoonBit は非対応**）。API-005 / API-006
  （ecosyste.ms）と API-003（libraries.io）は欠損の補完・照合に限定する。
- **付加指標**: API-008（GitHub GraphQL）でスター・最終コミット・Issue、API-017（Scorecard）で
  保守性、API-018（OSV）で脆弱性。ここが差別化要素。API-021（OSS Insight）はスター推移と
  スター作成者の内訳が取れるが、2026-11-21 の実測では提供が劣化しており採用を見送った。

レート制限が公式に明示されているのは API-003（60/分）、API-005／API-006（無料枠 300/時）、
API-007（60/時・5,000/時）、API-008（5,000 ポイント/時）、API-018（制限なし）。他は実測で
安全な間隔を決める必要がある。

料金は、無料が API-001／002／004／007〜010／012／016〜018、無料枠ありが API-005／006
（300 req/時、有償は 200 USD/月〜）、従量課金が API-011（月 1 TiB まで無料、以降 6.25 USD/TiB）、
登録要の無料が API-003。詳細は [01-apis.md](01-apis.md) の「料金」節。

### 2. 既存実装から学ぶのは「正確性」と「欠測の扱い」

3 実装はいずれも npm 専用で、健全性指標は扱っていない。学ぶべきは次の 3 点。

- **18 か月の壁**: npm downloads API は長期間を要求すると**黙って切り詰める**。TanStack の
  実測では単一リクエスト 585M に対しチャンク分割で 744M（27% の欠落）。取得は 500 日程度で
  分割し、取れた範囲を画面に明示する。
- **実測と推定を分ける**: npm.bet は補間値を `reportedDownloads` と別フィールドで保持し、
  共有 URL・埋め込み・OG にも設定を引き回す。libtrends の `meta.missing` も同じ思想。
- **外向きと内向きの両方のレート制限**: 取得側はチャンク間待機と並列度制御、公開側は
  IP 単位のスライディングウィンドウ（npm-trend-clone は 100 req/10 秒）。

### 3. 健全性は「EOL を足切り、脆弱性を深刻度、保守性を連続値」で扱う

- uzomuzo（v0.10.0）は `verdict`（ok / caution / replace / review）、`lifecycle`
  （active / stalled / eol）、`build_integrity`（0〜10）と**判定理由**を返す。実測では
  `request@2.88.2` が `replace` / `EOL-Confirmed`、`express@4.18.2` が `ok` / `Active`。
- **MoonBit は uzomuzo で判定できない**（実測: `error: package not found in deps.dev`、
  verdict は `review`、Scorecard も 404）。MoonBit は最終リリース日・yanked・ビルド状態・
  被依存から代替指標を組み立て、uzomuzo 判定とは別枠で表示する。
- vuls は OS・サーバ寄りのスキャナで、ライブラリ横断の脆弱性ソースには向かない。libtrends では
  レジストリ横断の脆弱性は API-018（OSV）を主とし、vuls は「利用環境の観測」に限定する
  （この VM には未導入のため、実出力での検証は未実施）。
- 合成スコアは初期実装では作らず、軸別に表示する。EOL は他の指標で覆さない。

### 4. 設計の要点

- 主キーは **PURL**。`identity` / `popularity` / `dependency` / `maintenance` / `security` /
  `meta` の 6 名前空間でスナップショットを保持する。`ecosystem` は npm / pypi / cargo / go / moonbit。
- 欠損は `null`、判定不能は `unknown`。**0 と欠損を混同しない**。
- 収集は「アダプタ → 生レスポンス保存 → 正規化 → 評価 → スナップショット DB → BFF → UI」の一方向。
  生レスポンスを残すことで、正規化ロジック変更時に再取得せず再処理できる。
- 画面はトップ（トレンド）／パッケージ詳細／比較／依存グラフ／健全性レポート／指標の説明。
  詳細画面には各値の取得元 API ID と取得日時を出す。

## 未決事項

- **スコア合成の是非と重み**: 軸別表示のままにするか、単一の健全性スコアを出すか。
  実データでの検証方法も含めて未決定。
- **対象カタログの作り方**: 「人気上位 N 件」の N と選定基準。レジストリ間の偏りをどう扱うか。
- **PyPI のダウンロード指標**: 非公式の API-010 で始めるか、最初から API-011（BigQuery）に
  投資するか。
- **ダウンロード数を持たないエコシステム**（Go: API-016、MoonBit: API-020 は累計のみ）の
  人気度を、被依存数だけで比較してよいか。
- **レート制限が不明な API の運用間隔**: API-004 / API-009 / API-012 / API-016 / API-017 /
  API-020 は公式に数値がない。実測で決める必要がある。
- **料金の上限管理**: ecosyste.ms の無料枠（300 req/時）と BigQuery の従量課金を、
  収集規模に対してどう抑えるか。無料枠を超える前提で予算を置くかも未定。
- **vuls の組み込み範囲**: 利用環境の観測機能を初期リリースに含めるか。実機検証も未実施。
- **OSS Insight（API-021）の復旧確認**: トレンドと stargazers 系が 2026-11-21 の実測で
  提供劣化していた。スター推移の取得先として使うには、定期的に実測して復旧を
  確認する必要がある。
- **データ鮮度の表示仕様**: 取得失敗時に古い値をどう見せるか（鮮度バッジの設計）。
- **利用条件の確認**: ecosyste.ms の商用利用条件、libraries.io のキー発行条件、
  crates.io のレート制限（自動取得では本文を取得できなかった）。

## 出典

各資料の末尾に個別の出典をまとめている。横断的に使った主な一次情報は次のとおり。

- npm downloads API: https://github.com/npm/registry/blob/master/docs/download-counts.md
- npm registry API: https://github.com/npm/registry/blob/master/docs/REGISTRY-API.md
- deps.dev API v3: https://docs.deps.dev/api/v3/
- ecosyste.ms: https://docs.ecosyste.ms/ （料金: https://ecosyste.ms/pricing）
- libraries.io API: https://libraries.io/api
- GitHub REST レート制限: https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api
- GitHub GraphQL レート制限: https://docs.github.com/en/graphql/overview/rate-limits-and-node-limits-for-the-graphql-api
- PyPI JSON API: https://docs.pypi.org/api/json/ （料金: https://cloud.google.com/bigquery/pricing）
- crates.io データアクセス: https://crates.io/data-access
- Go module proxy: https://go.dev/ref/mod#goproxy-protocol
- OpenSSF Scorecard: https://securityscorecards.dev/
- OSV.dev API: https://google.github.io/osv.dev/api/
- uzomuzo: https://github.com/future-architect/uzomuzo-oss
- mooncakes.io（MoonBit）: https://mooncakes.io/
- OSS Insight Public API: https://ossinsight.io/docs/api
- Simple Icons: https://simpleicons.org/
- Devicon: https://devicon.dev/
- vuls: https://vuls.io/
- TanStack 設計ブログ: https://tanstack.com/blog/npm-stats-the-right-way
