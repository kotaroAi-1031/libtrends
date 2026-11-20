# ライブラリ集計 API の網羅調査

## 調査の範囲と方法

libtrends が扱う指標（ダウンロード数・スター数・人気度・依存関係・保守性・脆弱性）を、
どの API から取得できるかを横断的に整理する。対象は「パッケージレジストリの公式 API」と
「複数レジストリを横断集約する API」の二層で、最後に GitHub のようなソースホスティング API と
セキュリティ系 API を加える。

### 対象言語と対応レジストリ

対象は次の 6 言語に限定する。TypeScript と JavaScript は同じ npm レジストリを共有するため、
レジストリは 5 つになる。

| 言語 | レジストリ | 担当 API | ダウンロード数の提供 |
| --- | --- | --- | --- |
| Go | proxy.golang.org | API-016 | なし |
| TypeScript / JavaScript | registry.npmjs.org | API-001（統計）、API-002（メタデータ） | あり（日次〜任意期間） |
| MoonBit | mooncakes.io | API-020 | 累計のみ（時系列なし） |
| Rust | crates.io | API-012 | あり（累計・直近90日） |
| Python | pypi.org | API-009（メタデータ）、API-010／API-011（統計） | 公式 JSON にはなし。別経路で取得 |

RubyGems（Ruby）、Packagist（PHP）、Maven Central（Java 系）、NuGet（.NET）は対象言語外のため
調査対象から外した（「対象外としたもの」参照）。

### 確認方法

- 公式ドキュメントの記述を一次情報として採用し、出典 URL を付す。
- 実際にリクエストを送り、HTTP ステータスとレスポンスの形を確認した（2026-11 時点）。
  実測した事実は本文中に「実測」と明記する。
- ドキュメントに記載が見当たらない項目は空欄にせず「不明（要確認）」と書く。

## API一覧

| ID | API | 提供指標 | 認証 | レート制限 | 出典 |
| --- | --- | --- | --- | --- | --- |
| API-001 | npm downloads API（api.npmjs.org） | 期間別ダウンロード数（日次・週次・月次・任意期間、パッケージ単位／バージョン単位は直近7日のみ）、複数パッケージの一括取得 | 不要 | 不明（要確認。公式ドキュメントに上限の記載なし。一括は128パッケージ・365日まで、通常取得は18か月までというデータ範囲の制限のみ） | https://github.com/npm/registry/blob/master/docs/download-counts.md |
| API-002 | npm registry API（registry.npmjs.org） | パッケージメタデータ（バージョン一覧、公開日時、依存関係、ライセンス、メンテナ、リポジトリURL、dist-tags） | 読み取りは不要（公開は書き込みにトークン） | 不明（要確認） | https://github.com/npm/registry/blob/master/docs/REGISTRY-API.md |
| API-003 | libraries.io API | 依存関係（直接・間接）、依存先数、スター数、フォーク数、コントリビュータ数、リリース頻度、ライセンス | 要（`api_key` クエリパラメータ。未指定でも一部応答する場合があるが公式には必須） | 60 リクエスト/分（APIキー単位、超過で 429） | https://libraries.io/api |
| API-004 | deps.dev API v3 | パッケージ・バージョンのメタデータ、依存関係（直接／間接）、被依存（dependents）、要求バージョン制約、プロジェクト情報、アドバイザリ（OSV由来）、バッチ照会 | 不要 | 不明（要確認。公式ドキュメントに上限の記載なし） | https://docs.deps.dev/api/v3/ |
| API-005 | ecosyste.ms packages API | レジストリ横断のパッケージメタデータ（ダウンロード数、依存数、ライセンス、リポジトリURL、出版日時、保守状況の要約） | 不要（大規模利用は有償プラン） | 不明（要確認。未認証は二段階の制限、有償プランで緩和） | https://docs.ecosyste.ms/ |
| API-006 | ecosyste.ms repos API | リポジトリ側の指標（スター数、フォーク数、最終コミット、コントリビュータ、Issue/PR 統計、依存パッケージとの対応） | 不要（大規模利用は有償プラン） | 不明（要確認。API-005 と同一の制限体系） | https://docs.ecosyste.ms/ |
| API-007 | GitHub REST API | スター数、フォーク数、Watcher、Issue/PR、コミット履歴、リリース、コントリビュータ、言語構成、Advisory（GHSA） | 任意（トークンで上限が上がる） | 未認証 60 リクエスト/時、認証時 5,000 リクエスト/時（実測で未認証 `core.limit=60` を確認） | https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api |
| API-008 | GitHub GraphQL API | API-007 と同等の指標を1リクエストで複数リポジトリ分まとめて取得、依存関係グラフ | 要（トークン） | 5,000 ポイント/時（ポイント制。クエリの複雑度で消費） | https://docs.github.com/en/graphql/overview/rate-limits-and-node-limits-for-the-graphql-api |
| API-009 | PyPI JSON API | パッケージメタデータ（バージョン、公開日時、ライセンス、分類子、作者、プロジェクトURL、requires_dist）、ファイル一覧 | 不要 | 不明（要確認） | https://docs.pypi.org/api/json/ |
| API-010 | pypistats.org API | PyPI のダウンロード数（日次・週次・月次、Python バージョン別・OS 別・ミラー別） | 不要 | 不明（要確認。非公式のサードパーティサービス） | https://pypistats.org/api |
| API-011 | PyPI BigQuery 公開データセット | PyPI の生ダウンロードログ（ファイル単位、日次バッチ）を SQL で集計 | 要（Google Cloud 認証・プロジェクト） | クエリ課金（スキャン量に応じた従量。無料枠の範囲は要確認） | https://docs.pypi.org/api/bigquery/ |
| API-012 | crates.io API | クレートメタデータ（バージョン、公開日時、ライセンス、依存関係、被依存、キーワード、カテゴリ）、累計・期間別ダウンロード数 | 読み取りは不要（User-Agent 必須） | 不明（要確認。公式データアクセスページにポリシー記載があるが、自動取得では本文を取得できなかった） | https://crates.io/data-access |
| API-016 | Go module proxy（proxy.golang.org） | モジュールのバージョン一覧（`@v/list`）、最新版（`@latest`）、バージョン情報（`.info`）、`go.mod`、zip。ダウンロード数は提供しない | 不要 | 不明（要確認） | https://go.dev/ref/mod#goproxy-protocol |
| API-017 | OpenSSF Scorecard API | リポジトリの健全性スコア（0〜10）とチェック項目別スコア（保守性、CI、コードレビュー、依存更新、脆弱性、ライセンス、署名など） | 不要 | 不明（要確認） | https://securityscorecards.dev/ |
| API-018 | OSV.dev API | 既知脆弱性（OSV 形式）。パッケージ・バージョン指定のクエリ、バッチクエリ、ID 直接取得 | 不要 | 明示的な上限なし（公式に "Currently there are no limits" と記載。レスポンスは HTTP/1.1 で 32MiB 上限） | https://google.github.io/osv.dev/api/ |
| API-020 | mooncakes.io registry API（MoonBit） | モジュール一覧、最新版・全バージョン（yanked 含む）、累計ダウンロード数、被依存モジュール一覧、ライセンス、リポジトリURL、キーワード、説明、ビルド状態、全体統計（総モジュール数・総ダウンロード数）。時系列のダウンロード推移と直接依存の一覧は提供しない | 読み取りは不要（API 自体が未文書化。公開には API トークン） | 不明（要確認。レスポンスに RateLimit 系ヘッダなし、公式ドキュメントなし） | https://mooncakes.io/ |

## 個別の補足

### API-001 npm downloads API（TypeScript / JavaScript）

- エンドポイントは `/downloads/point/{period}[/{package}]` と `/downloads/range/{period}[/{package}]`。
  期間は `last-day` / `last-week` / `last-month` と `YYYY-MM-DD:YYYY-MM-DD` 形式。
- 一括取得はカンマ区切りで最大 128 パッケージ、最大 365 日。スコープ付きパッケージは一括取得に未対応。
- バージョン別ダウンロード数は直近 7 日分のみ。データの最古は 2015-01-10。
- 実測: `GET https://api.npmjs.org/downloads/point/last-week/react` は
  `{"downloads":273970923,"start":"2026-11-12","end":"2026-11-18","package":"react"}` を返し、
  レート制限に関するヘッダは付かなかった。
- ダウンロード数は「インストール回数」ではなく「レジストリからの tarball 取得回数」であり、
  CI・ミラー・ボットの影響を受ける。指標の意味づけは 04-design-draft.md で扱う。

### API-002 npm registry API（TypeScript / JavaScript）

- パッケージドキュメント（`/{package}`）に全バージョンのメタデータが入るため、1 リクエストで
  バージョン履歴・依存関係・公開日時が取れる。ペイロードは大きく、人気パッケージでは数 MB になる。
- `Accept: application/vnd.npm.install-v1+json` を指定するとインストール用の縮小版（abbreviated）が返る。
- 実測: `GET https://registry.npmjs.org/react/latest` は 200 で最新版メタデータを返した。

### API-003 libraries.io

- ドキュメント上は全リクエストに `api_key` が必須で、上限は 60 リクエスト/分。
- 実測: `GET https://libraries.io/api/npm/react` はキーなしで 200 を返した。ただしこれは
  公式ドキュメントの記述と矛盾するため、キーなし運用を前提にしない。
- 依存関係の集約と「依存先数（dependents）」が強み。一方でデータ更新頻度はレジストリ直取得より遅い。
- 対象 6 言語のうち、MoonBit のカバレッジは未確認（プラットフォーム一覧に mooncakes が
  含まれるかを実装前に確認する）。

### API-004 deps.dev（対象 6 言語のうち 4 エコシステム）

- エンドポイントは `/v3/systems/{system}/packages/{name}`、`/v3/systems/{system}/packages/{name}/versions/{version}`、
  `/v3/projects/{id}`、`/v3/query`（バッチ）、`/v3/advisories/{id}`。
- 対応システムは npm / PyPI / Go / Cargo / Maven / NuGet。**MoonBit は非対応**で、
  実測では `GET /v3/systems/moonbit/packages/moonbitlang%2Fx` が 404 `package not found` を返した。
- 実測では npm / PyPI / Go / Cargo の 4 システムが 200。Go は `github.com%2Fgorilla%2Fmux` のように
  スラッシュをエンコードして渡す。
- 依存関係は「宣言された制約」と「解決済みの実依存」の両方を返す点が、単純なメタデータ API との差。
- Google が運用しており、OSS 向けに無償提供されている。

### API-005 / API-006 ecosyste.ms

- レジストリ横断の「パッケージ」と「リポジトリ」を別サービスとして提供する点が特徴。
  パッケージ側からリポジトリ側へ、リポジトリ側から依存パッケージへ辿れる。
- 実測: `https://packages.ecosyste.ms/api/v1/registries/npmjs.org/packages/react` は 200 で
  パッケージ情報を返した。`repos.ecosyste.ms` は `facebook/react` では 404 だったが、
  これはリポジトリが `react/react` へ移転しているためで、API の不具合ではない。
- 対象 6 言語では npm / PyPI / crates.io / Go を主要な照合先とする。MoonBit のレジストリが
  収録されているかは未確認（要確認）。
- 大規模利用は有償プランの領域。商用利用の条件はドキュメントの Licences / Commercial use を参照。

### API-007 / API-008 GitHub

- REST は未認証 60 リクエスト/時。実測時点で `core.remaining` は 0 で、未認証の運用は現実的でない。
- GraphQL はポイント制で、1 クエリで複数リポジトリのスター・Issue・最終コミットをまとめて取れるため、
  大量リポジトリの定期更新に向く。
- 6 言語すべてで、パッケージのリポジトリ URL（npm の `repository`、PyPI の `project_urls`、
  crates.io の `repository`、Go の `go.mod` のモジュールパス、mooncakes の `repository`）から
  GitHub リポジトリを特定できる。スター数はこの経路で取る。
- スター数は「人気度」の代理指標として最も一般的だが、獲得時点の話題性に強く依存し、
  現在の利用実態を反映しない。

### API-009 / API-010 / API-011 PyPI（Python）

- PyPI の JSON API にダウンロード数は含まれない。ダウンロード指標は別経路が必要。
- pypistats.org は PyPI の BigQuery データを集計したサードパーティサービスで、公式 API ではない。
  手軽だが可用性の保証はなく、一次情報としては API-011 が原資料。
- BigQuery 公開データセットはファイル単位の生ログで、集計の自由度が高い反面、
  クエリ課金と GCP 認証が前提になる。

### API-012 crates.io（Rust）

- User-Agent の設定が求められる。`/api/v1/crates/{name}` はクレート情報と
  `downloads`（累計）・`recent_downloads`（直近90日）を含む。
- 実測: `https://crates.io/api/v1/crates/serde` は 200 でクレート情報を返した。
- データアクセスのポリシーページは自動取得で本文が取れなかった（403／JS チャレンジ）。
  レート制限の数値は実装前にブラウザで確認する。

### API-016 Go module proxy（Go）

- `@v/list`、`@latest`、`@v/{version}.info`、`@v/{version}.mod`、`@v/{version}.zip` を提供。
  実測: `proxy.golang.org/github.com/gin-gonic/gin/@v/list` はバージョン一覧を返した。
- ダウンロード数は提供しない。Go エコシステムは「利用されているか」を proxy のログからは測れないため、
  被依存（API-004 の dependents）で補う。
- `.mod` を取得すれば直接依存を解析できるため、依存グラフの構築は自前で可能。

### API-017 OpenSSF Scorecard

- リポジトリ単位のスコアで、チェック項目に Maintained（保守状況）、Code-Review、CI-Tests、
  Dependency-Update-Tool、Vulnerabilities、License、Signed-Releases などがある。
- 実測: `api.securityscorecards.dev/projects/github.com/facebook/react` は 200 でスコアと
  チェック項目を返した（Scorecard v5.6.0）。
- **MoonBit の主要リポジトリは公開データセットに含まれていない**。実測では
  `github.com/moonbitlang/x`、`github.com/moonbitlang/core`、`github.com/moonbitlang/moon` の
  いずれも 404 を返した（リポジトリ自体は存在する）。MoonBit は Scorecard 依存の指標を
  欠損として扱う必要がある。
- 「保守性」を単一の数値で表せる稀有な公開指標。ただしスコアはリポジトリの設定状況の評価であり、
  利用者数や品質そのものではない。

### API-018 OSV.dev

- パッケージ・エコシステム・バージョンを指定して脆弱性を引けるため、レジストリ横断の
  脆弱性指標にそのまま使える。バッチ API があるので多数パッケージの定期チェックに向く。
- エコシステム名は npm / PyPI / crates.io / Go などを指定する。MoonBit のエコシステム名が
  登録されているかは未確認（要確認）。
- 公式にレート制限なしと明記されている点は、他の API と比べて運用しやすい。

### API-020 mooncakes.io（MoonBit）

MoonBit の公式パッケージレジストリは mooncakes.io。**公開 API ドキュメントは存在しない**
（サイトのドキュメントページに API 仕様の記載がなく、バックエンドのリポジトリも公開されていない）。
以下はフロントエンドのバンドルから端点を特定し、実リクエストで挙動を確認した結果である。
仕様が予告なく変わる前提で扱う。

| 端点 | 返すもの | 実測 |
| --- | --- | --- |
| `GET /api/v0/modules` | 全モジュールの一覧（名前・最新版・ライセンス・リポジトリ・キーワード・説明・作成日時）。ダウンロード数は含まない | 200、3,875 件、約 1.4MB |
| `GET /api/v0/modules?recent=N` | 最近更新されたモジュール（`{"recent":[...]}` 形式） | 200 |
| `GET /api/v0/modules/statistics` | 全体統計 `{"total_modules":3875,"total_packages":34886,"total_lines":63849117,"total_downloads":12103826}` | 200 |
| `GET /api/v0/manifest/{author}/{name}` | モジュール詳細。`downloads`（累計）、`dependents`（被依存モジュールとバージョン）、`versions`（yanked フラグ付き）、`latest_version`、`build_status`、`metadata`（ライセンス・リポジトリ・説明・キーワード・作成日時） | 200、`?version=` は無視され常に最新版 |
| `GET /api/v0/modules/{author}/{name}` | manifest から `downloads` を除いたもの | 200 |
| `GET /api/v0/search?kw={語}&limit={n}` | 検索結果。各件に `downloads` を含む | 200、既定 20 件、`limit=100` まで確認。`offset` は無視された |
| `GET /api/v0/skills` | MoonBit のスキル（エージェント向け定義） | 未確認 |

分かった制約。

- **時系列のダウンロード推移は提供しない**。フロントエンドにも推移グラフ用の端点がなく、
  `downloads` は累計値のみ。libtrends で推移を出すには日次でスナップショットを保存する必要がある。
- **直接依存の一覧を返す端点がない**。`dependents`（被依存）は取れるが、そのモジュールが
  何に依存しているかは API からは取れない。`moon.mod.json` を配布物から読むか、
  リポジトリを解析する必要がある（配布物の取得経路は未確認）。
- 認証は読み取り不要。公開（publish）には API トークン（`/api/v0/api_token/create`）を使う。
- レート制限は不明。レスポンスヘッダに `RateLimit-*` 系は付かず、公式の記載もない。
- 代替のデータ源として、レジストリのインデックスが GitHub リポジトリ
  （`moonbitlang/mooncakes.io-index`）で公開されている。ただし中身は
  `{"name","version","checksum"}` のみで、指標は含まない。

## 比較と選定

### 三つの層に分けて捉える

1. **一次ソース（レジストリ公式）**: API-001、API-002（npm）、API-009（PyPI）、API-012（crates.io）、
   API-016（Go proxy）、API-020（mooncakes.io）。各言語の「正」であり、更新が最も早い。
   一方でレスポンス形式がバラバラで、ダウンロード指標の有無も揃わない
   （Go は提供せず、MoonBit は累計のみ、PyPI は公式 JSON に無い）。
2. **横断集約**: API-004（deps.dev）、API-005／API-006（ecosyste.ms）、API-003（libraries.io）。
   共通スキーマで複数エコシステムを扱えるため、libtrends の土台に向く。ただし deps.dev は
   MoonBit 非対応で、更新頻度とカバレッジはレジストリ直取得に劣る。
3. **付加指標**: API-007／API-008（GitHub）、API-017（Scorecard）、API-018（OSV）。
   人気度・保守性・脆弱性はレジストリからは取れないため、この層が実質的な差別化要素になる。

### 6 言語のカバレッジ

| 指標 | TypeScript / JavaScript | Python | Rust | Go | MoonBit |
| --- | --- | --- | --- | --- | --- |
| メタデータ | API-002 | API-009 | API-012 | API-016 | API-020 |
| ダウンロード（期間別） | API-001 | API-010（または API-011） | API-012（累計・90日） | 提供なし | 提供なし（累計のみ） |
| 被依存 | API-004 | API-004 | API-004 | API-004 | API-020 `dependents` |
| 直接依存 | API-004 / API-002 | API-004 / API-009 | API-004 / API-012 | 自前解析（`.mod`） | 取得不可（要確認） |
| スター | API-007／API-008 | 同左 | 同左 | 同左 | 同左（公開データセットの有無は要確認） |
| 保守性スコア | API-017 | API-017 | API-017 | API-017 | 取得不可（実測 404） |
| 脆弱性 | API-018 | API-018 | API-018 | API-018 | 対応未確認 |

### 推奨スタック（案）

| 役割 | 採用候補 | 理由 |
| --- | --- | --- |
| パッケージ同定・メタデータの基準 | API-004 deps.dev | 認証不要、npm / PyPI / Go / Cargo を同一スキーマで扱える。依存と被依存も取れる |
| MoonBit のメタデータ | API-020 mooncakes.io | deps.dev が非対応のため唯一の一次ソース。仕様が未文書化な点に留意 |
| ダウンロード数の一次取得 | API-001（npm）、API-010／API-011（PyPI）、API-012（crates.io）、API-020（MoonBit・累計のみ） | 期間別の推移を出せるのは npm と PyPI、crates.io は累計と直近90日 |
| リポジトリ指標 | API-008 GitHub GraphQL（補助として API-006） | スター・最終コミット・Issue をまとめて取得でき、クエリ数あたりの効率がよい |
| 保守性スコア | API-017 Scorecard | チェック項目別の内訳があり、根拠を画面に出せる。MoonBit は欠損 |
| 脆弱性 | API-018 OSV | レート制限なし、レジストリ横断、バッチ対応 |
| 欠損補完・照合 | API-003 libraries.io、API-005 ecosyste.ms | 依存関係やスターの突き合わせに使う。APIキーと有償枠の制約に注意 |

### 選定の判断基準

- **認証の少なさ**を優先する。API-004、API-018、API-017、API-020 はキーなしで使え、初期構築の障壁が低い。
- **レート制限が明示されているもの**を優先する。API-003（60/分）、API-007（60/時・5,000/時）、
  API-008（5,000 ポイント/時）、API-018（制限なし）は設計に織り込める。上限が不明な API は
  バッチ間隔を保守的に取り、429／5xx を見て動的に絞る。
- **一次情報かどうか**を指標の信頼度に反映する。ダウンロード数はレジストリ直取得を正とし、
  集約 API の値は照合用とする。
- **指標が欠けるエコシステムを明示する**。Go と MoonBit は期間別ダウンロード数を持たないため、
  横並び比較では「データなし」と表示する（0 として描かない）。

### 使わない／限定利用とするもの

- API-011 BigQuery は自由度が高いが、初期段階では課金と運用の重さが見合わない。PyPI の
  ダウンロード推移は API-010 で代替し、必要になった時点で移行する。
- API-003 libraries.io はキー必須で上限も低い。常用せず、欠損時の照合に限定する。

## 対象外としたもの

対象 6 言語の外にあるため、次のレジストリ API は調査対象から外した。ID は欠番として残す
（既存の参照を壊さないため、番号の振り直しはしない）。

| ID | API | 対象外の理由 |
| --- | --- | --- |
| API-013 | RubyGems.org API | Ruby は対象言語外 |
| API-014 | Packagist API | PHP は対象言語外 |
| API-015 | Maven Central Search API | Java / Kotlin / Scala などは対象言語外 |
| API-019 | NuGet V3 API | .NET（C# / F#）は対象言語外 |

将来これらを追加する場合も、指標モデル（04-design-draft.md）は共通スキーマのまま拡張できる。
ダウンロード数の定義が異なる点（Maven と NuGet の差）だけは、追加時に再確認が必要になる。

## 出典

- npm downloads API: https://github.com/npm/registry/blob/master/docs/download-counts.md
- npm registry API: https://github.com/npm/registry/blob/master/docs/REGISTRY-API.md
- libraries.io API: https://libraries.io/api
- deps.dev API v3: https://docs.deps.dev/api/v3/
- ecosyste.ms ドキュメント: https://docs.ecosyste.ms/
- ecosyste.ms レート制限: https://docs.ecosyste.ms/docs/usage/rate-limits/
- GitHub REST レート制限: https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api
- GitHub GraphQL レート制限: https://docs.github.com/en/graphql/overview/rate-limits-and-node-limits-for-the-graphql-api
- PyPI JSON API: https://docs.pypi.org/api/json/
- pypistats.org API: https://pypistats.org/api
- PyPI BigQuery: https://docs.pypi.org/api/bigquery/
- crates.io データアクセス: https://crates.io/data-access
- Go module proxy: https://go.dev/ref/mod#goproxy-protocol
- OpenSSF Scorecard: https://securityscorecards.dev/
- OSV.dev API: https://google.github.io/osv.dev/api/
- mooncakes.io（MoonBit レジストリのポータル。API 仕様の記載はなし）: https://mooncakes.io/
- mooncakes.io フロントエンドのリポジトリ: https://github.com/moonbitlang/mooncakes.io
- mooncakes.io レジストリインデックス（name / version / checksum のみ）: https://github.com/moonbitlang/mooncakes.io-index
