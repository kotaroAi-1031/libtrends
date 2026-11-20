# 健全性指標への組み込み（uzomuzo / vuls）

## 概要

ダウンロード数とスター数は「どれだけ使われているか」しか示さない。ライブラリ選定で本当に
知りたいのは「この先も保守されるか」「既知の脆弱性を抱えていないか」である。本資料では、
その二つを外部ツールの出力から取り込み、libtrends の共通指標に変換する設計をまとめる。

対象ツールは次の二つ。

| ツール | 役割 | 本調査での扱い |
| --- | --- | --- |
| uzomuzo | EOL・放棄（メンテナンス終了）検出 CLI。PURL を入力にライフサイクル判定を返す | この VM に導入済み（v0.10.0）。実際に実行して出力を確認した |
| vuls | サーバ・OS・コンテナ向けの脆弱性スキャナ（agentless） | この VM には未導入。設計は公式ドキュメントの出力仕様に基づく |

両者は守備範囲が異なる。uzomuzo は「パッケージが生きているか」を、vuls は「実際に配備された
環境にどんな脆弱性が残っているか」を答える。libtrends では前者をライブラリ単位の指標に、
後者を利用環境側の観測値として扱う（理由は「指標モデルへの対応付け」で述べる）。

対象言語は Go / TypeScript / JavaScript / MoonBit / Rust / Python の 6 言語。
uzomuzo は deps.dev を参照するため、deps.dev が扱わない **MoonBit は判定できない**。
この欠落は「uzomuzo」節の末尾で実測結果とあわせて扱う。

## uzomuzo

### 何をするツールか

- OSS 依存の「健康診断」を行う Go 製 CLI。バージョンは v0.10.0。
- 入力は PURL（`pkg:npm/express@4.18.2`）、PURL のリストファイル、標準入力、SBOM（CycloneDX / SPDX）、
  あるいは GitHub のリポジトリ URL。
- 出力は「判定（verdict）」「ライフサイクル」「ビルド整合性」の三段構えで、JSON にもできる。
- 判定に使う材料は、リポジトリの最終リリース・コミット状況、依存数、OpenSSF Scorecard の
  スコアなど。ツール自体が deps.dev や Scorecard のビューア URL を出力に含めている。
- 公式 README は「16,000 の実運用パッケージを分析したところ 48% にライフサイクルリスクがあった」
  と述べており、CVE が付かないまま放置される依存の多さを根拠にしている。

### 実際の出力（実測）

`uzomuzo scan pkg:npm/express@4.18.2` の要点。

```text
STATUS     PURL                    LIFECYCLE  BUILD
✅ ok       pkg:npm/express@4.18.2  Active     Hardened 9.5

│ Recent Stable Release: true
│ Maintained Score: 10/10
│ 69687 stars
│ Used by: 2182 packages
│ Depends on: 31 direct, 39 transitive
│ Scorecard Overall: 8.5/10  Maintained: 10.0/10
│ Stable: 5.3.0 (2026-10-09)
│ Requested: 4.18.2 (2022-10-08)
```

`--format json` の出力（抜粋）。

```json
{
  "summary": { "total": 1, "ok": 1, "caution": 0, "replace": 0, "review": 0,
    "build_integrity": { "hardened": 1, "moderate": 0, "weak": 0, "ungraded": 0 } },
  "packages": [{
    "purl": "pkg:npm/express@4.18.2",
    "verdict": "ok",
    "lifecycle": "Active",
    "build_integrity": "Hardened",
    "build_integrity_score": 9.5,
    "repo_url": "https://github.com/expressjs/express",
    "archived": false,
    "disabled": false,
    "overall_score": 8.5,
    "dependent_count": 2182,
    "stable_version": "5.3.0",
    "project_license": "MIT",
    "version_license": "MIT",
    "reason": "Actively maintained with recent releases"
  }]
}
```

停止・EOL 側の挙動も確認した。

```text
pkg:npm/request@2.88.2   verdict=replace   lifecycle=EOL-Confirmed   reason="Deprecated in npm registry"
pkg:pypi/django@2.2      verdict=caution   lifecycle=Stalled         reason="Maintained but no commit data"
```

`request` は npm レジストリの deprecated フラグを根拠に EOL 確定と判定される。`django@2.2` は
「メンテナンスは続いているがコミット情報が取れない」ため caution に落ちる。つまり判定は
**根拠の種類によって確度が変わる**。

### 出力フィールドの意味

| フィールド | 値の例 | libtrends での解釈 |
| --- | --- | --- |
| `verdict` | `ok` / `caution` / `replace` / `review` | 総合判定。画面のバッジにそのまま使う |
| `lifecycle` | `Active` / `Stalled` / `EOL-Confirmed` | 保守状態。EOL は「使うべきでない」シグナル |
| `build_integrity` | `Hardened` / `Moderate` / `Weak` / `Ungraded` | サプライチェーン側の健全性（レビュー体制、CI、依存固定など） |
| `build_integrity_score` | 0〜10 | 上記の数値化。Scorecard の項目別スコアが元 |
| `overall_score` | 0〜10 | 総合スコア（Scorecard Overall 相当） |
| `dependent_count` | 2182 | 被依存パッケージ数。人気度の補助指標になる |
| `reason` | 文字列 | 判定根拠。UI で「なぜこの判定か」を出すために必須 |
| `archived` / `disabled` | 真偽値 | リポジトリのアーカイブ・無効化 |

`days_since_last_release` や `deprecated` は JSON に存在しない場合がある（実測では `None`）。
**ツールのバージョンによってフィールドの有無が変わる**前提で、取り込み側は欠損を許容する。

### 取り込み設計

1. 対象 PURL のリストを libtrends 側のカタログから生成する（レジストリ横断なので PURL が共通キー）。
2. `uzomuzo scan --file purls.txt --format json` をバッチ実行し、生 JSON をそのまま保存する
   （判定ロジックの変更を後から追跡できるようにするため、正規化前の原本を残す）。
3. 正規化層で `verdict` / `lifecycle` / スコア類 / `reason` を共通スキーマへ写像する。
4. `verdict` が `replace` のものだけをアラート対象にし、`caution` は「要注意」として弱く表示する。
5. 実行コストは 1 件あたり数秒（実測: 1 件で数秒、2 件で 10 秒未満）。数千件規模では
   並列度とキャッシュで調整する。

CI ゲート用途としては `--fail-on eol-confirmed` が用意されている。libtrends では自前の
スコアリングに使うが、同じ判定基準を「依存関係の監査」機能に流用できる。

### MoonBit は判定できない（実測）

uzomuzo は deps.dev のパッケージ情報を前提にするため、deps.dev が非対応の MoonBit では
判定が成立しない。実測（2026-11-20）は次のとおり。

```text
$ uzomuzo scan pkg:moonbit/moonbitlang/x@0.5.7 --format json
verdict    = review
lifecycle  = Review Needed
build_integrity = Ungraded (0)
error      = "package not found in deps.dev"
```

`review` は「判断できなかった」を意味し、健全であることを意味しない。Scorecard も
MoonBit の主要リポジトリを公開データセットに持たない（`github.com/moonbitlang/x` などは 404）。
したがって MoonBit では、次の一次情報から自前で保守性を組み立てる必要がある。

| 代替シグナル | 取得元 | 意味 |
| --- | --- | --- |
| 最終リリース日 | API-020 `versions` / `metadata.created_at` | 更新の止まり具合 |
| yanked の有無 | API-020 `yanked` / `versions[].yanked` | 撤回された版か |
| ビルド状態 | API-020 `build_status` | 公開時のビルド成否 |
| 被依存数 | API-020 `dependents` | 利用規模と保守の動機 |
| スター・最終コミット | API-007／API-008（リポジトリ URL 経由） | 開発の継続性 |

この代替判定は libtrends 側のロジックであり、uzomuzo の判定と同列には置かない。
画面では「uzomuzo 判定なし（MoonBit）」と明示し、代替指標を別枠で見せる。

## vuls

### 何をするツールか

- Linux / FreeBSD などのサーバ、Docker イメージ、WordPress、非 OS パッケージを対象に、
  既知脆弱性を検出する agentless スキャナ。
- 脆弱性データベースは NVD、OVAL、JVN などを取り込む（go-cve-dictionary / goval-dictionary などの
  周辺ツール群で供給）。
- レポートは `vuls report -format-json` で JSON、`-format-cyclonedx-json` で CycloneDX 形式で出せる。

**この VM には vuls が導入されていない**（`vuls` コマンドが存在しないことを確認）。したがって
以下の設計は公式ドキュメントの出力仕様に基づくもので、実出力での検証は未実施である。

### libtrends での位置づけ

vuls は「レジストリ全体の脆弱性データベース」ではなく「ある環境の現在の脆弱性」を出す。
そのため libtrends のライブラリ単位スコアの主ソースにはしない。用途は次の二つに限定する。

1. **利用環境の観測**: 利用者が自分の SBOM／構成を投入したとき、vuls の JSON を読み込んで
   「このプロジェクトが使っているバージョンのうち、いくつが脆弱か」を表示する。
2. **検証済み知見の還元**: 実際に悪用可能と判定された組み合わせ（OS パッケージとライブラリの
   共存など）を、ライブラリ側の注意書きとして蓄積する。

レジストリ横断の「既知脆弱性の有無」は OSV（[01-apis.md](01-apis.md) の API-018）で取得する。
OSV はパッケージ・エコシステム・バージョンで引けるため、ライブラリ単位の指標に向く。
vuls はその OSV 判定を「実環境で裏付ける」二次的な役割と整理する。

### 出力の取り込み設計（案）

`vuls report -format-json` の結果から、次の項目を正規化して取り込む想定。

| vuls 側の情報 | libtrends 側のフィールド | 用途 |
| --- | --- | --- |
| 対象ホスト／コンテナ | `environment.id` | 観測元の識別（公開指標には出さない） |
| パッケージ名・バージョン | `package.purl` へ変換 | ライブラリ指標との突合キー |
| CVE / アドバイザリ ID | `vuln.ids[]` | 詳細ページへのリンク |
| CVSS スコア・ベクタ | `vuln.cvss.score` / `vuln.cvss.vector` | 深刻度の並べ替え |
| 深刻度（critical / high / …） | `vuln.severity` | バッジ表示 |
| 修正版の有無 | `vuln.fixed_in` | 「更新で解決できるか」の提示 |
| 確度（confidence） | `vuln.confidence` | 誤検知の抑制。低確度はスコアに算入しない |

CycloneDX 出力があるため、SBOM を介した突合も可能。libtrends の取り込み口は
「JSON 直取り」と「CycloneDX 経由」の二系統を用意し、前者を主、後者を互換用とする。

## 指標モデルへの対応付け

### 健全性に関わる指標の対応表

[04-design-draft.md](04-design-draft.md) の共通スキーマに対して、どのツールがどの項目を埋めるか。

| 共通指標 | 定義 | 供給元 | 欠損時の扱い |
| --- | --- | --- | --- |
| `health.lifecycle` | 保守状態（active / stalled / eol） | uzomuzo `lifecycle` | `unknown` |
| `health.verdict` | 総合判定（ok / caution / replace / review） | uzomuzo `verdict` | `unknown` |
| `health.eol` | EOL 確定か | uzomuzo（`EOL-Confirmed`） | `false` ではなく `unknown` |
| `health.maintained_score` | 保守状況のスコア（0〜10） | uzomuzo / Scorecard（API-017） | `null` |
| `health.build_integrity` | サプライチェーン健全性（0〜10） | uzomuzo `build_integrity_score` | `null` |
| `health.stars` | スター数 | GitHub（API-007 / API-008） | `null` |
| `health.dependents` | 被依存数 | deps.dev（API-004）、uzomuzo `dependent_count`、MoonBit は API-020 | `null` |
| `vuln.open_count` | 未修正の既知脆弱性数 | OSV（API-018）、vuls | `null` |
| `vuln.max_cvss` | 最大 CVSS | OSV、vuls | `null` |
| `vuln.fixed_available` | 修正版の有無 | OSV、vuls | `unknown` |

MoonBit だけは `health.maintained_score` と `health.build_integrity` が埋まらない
（uzomuzo が判定不能、Scorecard も未収録）。この 2 項目は `null` のままにし、
代替シグナル（最終リリース日・yanked・ビルド状態）を別フィールドで持たせる。

### 合成の考え方（案・推測を含む）

以下は本調査時点の設計案であり、重み付けは実データで検証していない（**推測**）。

- **EOL は足切り**: `health.eol == true` のライブラリは他指標に関わらず「非推奨」バッジを出す。
  ダウンロード数が多くても覆さない（例: `request` は今も広く使われているが EOL）。
- **脆弱性は深刻度で重み付け**: `vuln.max_cvss` を主、件数を従とする。確度の低い検出は除外。
- **保守性は連続値として扱う**: `maintained_score` と `build_integrity` を 0〜100 に正規化し、
  平均ではなく最小値寄り（低い方を強調）に合成する。片方しか無い場合は欠損として扱い、
  0 点と混同しない。
- **人気度は健全性と分離**: ダウンロード数・スターは「利用規模」、健全性は「継続性」。
  一つの総合点に混ぜると「人気だが EOL」を誤って高評価にするため、画面でも別軸で見せる。

### 運用上の注意

- uzomuzo の判定は外部サービス（deps.dev、Scorecard、レジストリ）に依存する。取得失敗時は
  `unknown` を返し、前回値を「古い値」として保持する（判定不能を ok にしない）。
- MoonBit は uzomuzo の判定対象外（`error: package not found in deps.dev`）。`review` を
  「問題なし」と読み替えないよう、UI 側で明示する。
- vuls は OS パッケージの脆弱性が中心で、言語エコシステムのライブラリ網羅性は OSV に劣る。
  スコアの主ソースにしない理由がここにある。
- どちらのツールも「根拠（reason / CVE ID）」を保持できる。スコアだけを保存すると説明できない
  画面になるため、根拠文字列と参照 URL を必ず併存させる。

## 出典

- uzomuzo 公式リポジトリ（README・ライセンス、Apache-2.0）: https://github.com/future-architect/uzomuzo-oss
- uzomuzo 実機出力: 本 VM の `~/workspace/tools/uzomuzo/uzomuzo` v0.10.0 を実行（2026-11-19、MoonBit の確認は 2026-11-20）
- vuls 公式サイト: https://vuls.io/
- vuls レポート（`-format-json` / `-format-cyclonedx-json`）: https://vuls.io/docs/en/usage-report.html
- vuls リポジトリ: https://github.com/future-architect/vuls
- OpenSSF Scorecard: https://securityscorecards.dev/
- OSV.dev API: https://google.github.io/osv.dev/api/
- deps.dev API: https://docs.deps.dev/api/v3/
- mooncakes.io（MoonBit レジストリ）: https://mooncakes.io/
