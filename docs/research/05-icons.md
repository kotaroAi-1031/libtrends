# アイコン取得元の調査

## 概要

libtrends の画面（一覧・パッケージ詳細のヘッダ・比較）では、パッケージと
GitHub リポジトリを識別するアイコンが要る。2026-11-21 に取得元を調査し、
候補の URL を実際にリクエストして取得できることを確認した（以下「実測」は
2026-11-21 の確認結果）。

結論を先に書く。パッケージ自体の公式アイコンは存在しない。レジストリの
メタデータにアイコンの項目が無いため、複数の取得元を優先順に組み合わせる
設計になる。言語・レジストリのロゴは Simple Icons と Devicon に揃って
いるが、MoonBit だけはどちらにも収録されていない。

## GitHub リポジトリのアイコン

GitHub には「リポジトリのアイコン」という公式項目がない。実質的な
アイコンは次の 2 つである。

- **オーナーのアバター**: `https://github.com/{owner}.png?size=64` で
  取得できる（実測: 200、PNG）。REST API（API-007）のレスポンスにも
  `owner.avatar_url` が含まれるため、リポジトリ情報を取得するついでに
  アバターの URL も得られる。正方形で、一覧のアイコンに向く。
- **リポジトリの OG 画像**:
  `https://opengraph.githubassets.com/1/{owner}/{repo}` で取得できる
  （実測: 200、PNG）。リポジトリのソーシャルプレビュー画像で横長のため、
  アイコンではなくカード表示やリンクプレビューに向く。

## パッケージのアイコン

npm レジストリと PyPI のメタデータには、アイコンやロゴの項目がない
（実測: npm の `react`、PyPI の `requests` のメタデータに該当キーなし）。
crates.io、Go module proxy、mooncakes.io の公式メタデータにも、アイコンの
項目は確認できない。したがって、パッケージのアイコンは次の優先順で
代替する。

1. **リポジトリのオーナーアバター**（GitHub）。大半のパッケージは
   レジストリのメタデータからリポジトリ URL が得られるため、この経路で
   賄える。
2. **ホームページの favicon**。パッケージのホームページのドメインから
   取得する。キー不要の取得サービスが 2 つある（いずれも実測: 200）。
   - DuckDuckGo: `https://icons.duckduckgo.com/ip3/{domain}.ico`
     （react.dev、crates.io で取得を確認）
   - Google: `https://www.google.com/s2/favicons?domain={domain}&sz=128`
     （react.dev で取得を確認）
   ただし万能ではなく、mooncakes.io と pkg.go.dev は DuckDuckGo 経由では
   取得できなかった（実測: 404）。
3. **ロゴ専門 API**: logo.dev（`https://img.logo.dev/{domain}`）は
   ドメインからロゴを返すが、トークンが要る有償サービスである。
   同種の Clearbit Logo API は 2025-12-08 に終了したため使えない。
4. **言語・レジストリのアイコン**（次節）。パッケージ固有の画像が
   取れない場合の受け皿にする。
5. **頭文字のプレースホルダ**。上記すべてが取れない場合に、パッケージ名の
   頭文字から生成する。

## 言語・レジストリのアイコン

言語とレジストリの公式ロゴは、アイコンライブラリから取得できる。

| 取得元 | 形式 | ライセンス | URL の形 |
| --- | --- | --- | --- |
| Simple Icons | 単色 SVG | CC0-1.0 | `https://cdn.simpleicons.org/{slug}` |
| Devicon | カラー SVG（original / plain / line 等の版あり） | MIT | `https://cdn.jsdelivr.net/gh/devicons/devicon/icons/{name}/{name}-original.svg` |
| Iconify | SVG（API で動的生成） | 収録セットごとに異なる | `https://api.iconify.design/{prefix}/{name}.svg` |

実測で取得を確認したもの:

- Simple Icons: npm、python、rust、go、typescript、javascript、github、
  pypi は 200。cratesio と moonbit は 404 で未収録。
- Devicon: go、python、rust、typescript、javascript の
  `-original.svg` は 200。moonbit は未収録。
- Iconify: Simple Icons や logos のセットを同じ URL 形式で引ける。
  moonbit はどちらのセットにも無く 404。

MoonBit だけは 3 つすべてに収録されていない。代替は GitHub の
moonbitlang 組織のアバター `https://github.com/moonbitlang.png`
（実測: 200）で、これは「GitHub リポジトリのアイコン」と同じ経路である。

## 役割の重なりと使い分け

取得元どうしで役割が重なる箇所が 3 つある。

- **Simple Icons と Devicon と Iconify**: 言語・ブランドのロゴという
  役割は 3 者で重なる。違いは、Simple Icons が単色でブランドの公式色を
  1 色に落とした版（CC0 でライセンスが最も軽い）、Devicon がカラーの
  公式ロゴ寄りの版（MIT）、Iconify が両者を含む複数セットへの集約 API
  （取得経路の重なりであり、中身は各セットそのもの）という点にある。
  混在させると同じ言語でも見た目が揃わないため、正は 1 つに決める。
- **favicon の DuckDuckGo と Google と logo.dev**: 「ドメインから
  ロゴ・favicon を引く」役割が重なる。DuckDuckGo と Google はキー不要で
  無料だが、解像度と収録範囲にばらつきがある。logo.dev は有償で
  トークンが要る代わりに、ロゴとしての品質と収録範囲が広い。
- **GitHub のアバターと OG 画像**: どちらも「リポジトリを表す画像」
  だが、正方形のアバターは一覧のアイコン用、横長の OG 画像はカード用と
  役割が分かれる。混用しない。

## libtrends での採用方針（案）

- 言語・レジストリのロゴは Simple Icons を正とする（CC0 で制約が
  最も少なく、CDN の URL が安定している）。カラー表示が要る画面では
  Devicon を使うが、1 画面内で混在させない。Iconify は取得経路を
  一本化したい場合の代替にとどめる。
- パッケージのアイコンは、オーナーアバター → favicon（DuckDuckGo を
  第一、Google を第二）→ 言語アイコン → 頭文字の順にフォールバック
  する。logo.dev は有償のため初期実装では使わない。
- MoonBit の言語アイコンは moonbitlang 組織のアバターで代替する。
  Simple Icons か Devicon に MoonBit が収録されたら差し替える。
- crates.io のレジストリアイコンは Simple Icons に無いため、当面は
  Rust のアイコン（rust）で代替するか、crates.io の favicon
  （DuckDuckGo 経由で取得可）を使う。

## 出典

- Simple Icons: https://simpleicons.org/ （CDN の使い方は
  https://www.npmjs.com/package/simple-icons ）
- Devicon: https://devicon.dev/ （リポジトリは
  https://github.com/devicons/devicon ）
- Iconify API: https://iconify.design/docs/api/svg.html
- GitHub REST API（owner.avatar_url）:
  https://docs.github.com/en/rest/repos/repos
- logo.dev（Clearbit Logo API の終了と移行先）:
  https://www.logo.dev/docs/migrations/clearbit
- DuckDuckGo favicon サービスと Google favicon サービスは公式ドキュメントが
  無く、URL の形と取得可否は 2026-11-21 の実測に基づく。
