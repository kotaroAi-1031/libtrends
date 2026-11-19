# 既存実装の分析（repomix 3 点）

## 対象と方法

`docs/research/references/` に置かれた repomix パック 3 点を読み、libtrends の設計に
流用できる知見と、真似すべきでない点を整理する。

| 資料 | 元リポジトリ | 規模の目安 | 位置づけ |
| --- | --- | --- | --- |
| `npm-bet.md` | https://github.com/vercel/npm.bet | 約 325KB / 約 11,400 行 | Vercel 製の npm ダウンロード可視化サイト |
| `tanstack-npm-stats.md` | https://github.com/TanStack/tanstack.com（`src/components/npm-stats/` ほか） | 約 309KB / 約 10,200 行 | TanStack 公式サイトの統計機能と設計ブログ |
| `npm-trend-clone.md` | https://github.com/laststance/npm-trend-clone | 約 321KB / 約 10,900 行 | npmtrends.com のモダンなクローン |

分析方法は静的読解のみ（実行はしていない）。リポジトリ全体ではなく、次を重点的に読んだ。

- データ取得層（server action、API route、server function、hooks）
- キャッシュとレート制限の実装
- 期間指定・集約（ビニング）・欠測値の扱い
- 画面構成と出力機能（埋め込み、共有、エクスポート）
- テストと品質管理の体制

repomix は「Git の変更回数が多いファイルほど後ろに並ぶ」形式で出力されており、
`## File: <path>` の見出しでファイル境界を判断した。UI コンポーネント（`components/ui/`）は
shadcn/ui の生成物が大半のため、分析対象から外している。

## 個別分析

### npm.bet（vercel-labs/npm.bet）

#### 何を作っているか

npm パッケージのダウンロード統計を可視化・比較するサイト。Next.js 16 / React 19 /
Recharts / Tailwind CSS 4 で構成され、状態は `nuqs` で URL に持つ。README には
「A beautiful, interactive npm package download statistics visualizer」とある。

#### データ取得

- `actions/package/get.ts` の server action が `https://api.npmjs.org/downloads/range/{period}/{package}`
  を直接叩く。
- 期間は `last-week` / `last-month` / `last-year` に加え、`last-2-years`・`last-5-years`・
  `all-time` を **日付範囲文字列に変換**して渡す（`all-time` は 2010-01-01 起点）。
- パッケージ検索は `https://registry.npmjs.com/-/v1/search`（README の API Integration 節）。

```typescript
const convertTimeRangeToAPIFormat = (timeRange: string): string => {
  if (timeRange === "last-week" || timeRange === "last-month" || timeRange === "last-year") {
    return timeRange;
  }
  if (timeRange.includes(":")) return timeRange;
  const today = new Date();
  switch (timeRange) {
    case "last-2-years": { /* 2 年前の日付を計算 */ }
    case "all-time": {
      const npmStart = new Date(2010, 0, 1);
      return `${formatDateForAPI(npmStart)}:${formatDateForAPI(today)}`;
    }
  }
};
```

#### キャッシュ

- Upstash Redis（`Redis.fromEnv()`）にキー `package:{package}:{apiTimeRange}` で保存。
- TTL は 3600 秒（1 時間）。開発環境ではキャッシュを読み書きしない。
- 書き込みは Next.js の `after()` でレスポンス後に実行し、レイテンシに乗せない。

```typescript
const CACHE_TTL_SECONDS = 3600;
// 参照
const cached = await redis.get<PackageData>(cacheKey);
if (cached) return cached;
// 取得後、レスポンスを待たせずに保存
after(() => { redis.setex(cacheKey, CACHE_TTL_SECONDS, data); });
```

#### 指標の見せ方

- 複数パッケージの同時比較、日／週／月の集約切替、期間切替。
- **ゼロ欠測の補間をオプトインで提供**している点が独特。`lib/download-data.ts` に
  `zeroModes = ["reported", "estimated"]` を持ち、連続 7 日までのゼロを「前後 3 週間の
  同じ曜日の平均」で補い、履歴が無ければ線形補間する。補間した点は中空マーカーで示し、
  ツールチップには実測値を残す。
- 補間設定は共有 URL・SVG 埋め込み・OG プレビューにも `zeroMode=estimated` として引き回す。
- PNG エクスポート、SVG 埋め込み（Markdown / HTML / 直 URL のコードスニペット生成）、
  OG 画像生成（`app/og/route.tsx`）、SVG エンドポイント（`app/svg/route.ts`）。

#### 設計の工夫

- **「ゼロは障害の証明ではない」という立場を UI とデータ構造の両方に埋め込んでいる**。
  実測値 `reportedDownloads` と補間値 `downloads` を別フィールドで保持するため、
  後から表示を切り替えても元データが壊れない。
- 変換ロジックを `lib/` の純粋関数に切り出し、`tests/fixtures/download-gaps.json` に
  実レスポンスを凍結して回帰テストしている（`pnpm test`）。
- 埋め込み・共有・OG で同じ設定を再現するため、状態を URL に集約している。

#### 弱点

- npm 専用。言語横断の抽象は持たない。
- `all-time` を単一リクエストで取る実装は、npm API の 18 か月上限に当たる
  （この挙動は TanStack の検証記事で数値付きで示されている。後述）。**サイレントに
  短い期間になる**ため、画面の「all-time」表示と実データの範囲がずれる可能性がある。
- キャッシュが Redis 前提で、Vercel 以外の環境では追加設定が要る。
- 認証を持たず、負荷対策はキャッシュのみ。

### TanStack NPM Stats

#### 何を作っているか

tanstack.com の `/stats` で、TanStack エコシステム 200 以上のパッケージのダウンロード数を
可視化する機能。設計の背景は同梱のブログ記事
`src/blog/npm-stats-the-right-way.md`（"How We Track Billions of Downloads: The NPM Stats API
Deep Dive"、2025-12-02、著者 Tanner Linsley）に詳しい。

#### 設計ブログの要点

1. **point と range は同じ数を返すが、履歴の扱いが違う**。短期間では両者一致（7 日、
   30 日、12 か月で差分 0.00%）。
2. **18 か月の壁**: 18 か月を超える期間を要求すると、API はエラーにせず**黙って切り詰める**。
   18 か月・24 か月・全期間のいずれを要求しても、返るのは 549 日分で同じ値になる。
3. **実測比較**: `@tanstack/react-query` の 2019-10-25 以降を単一リクエストで取ると
   585,291,892（549 日）。500 日ずつ 5 分割して取得すると 744,337,492（2,235 日）。
   **差分 1 億 5,900 万、全体の 27%** が単一リクエストでは欠落する。
4. **正解はチャンク分割**: 18 か月弱（500 日）ずつ区切って `range` を繰り返し呼び、
   日次値を合算する。チャンク間には 200ms の待機を入れ、リトライを付ける。
5. **運用**: 8 パッケージ並列、DB キャッシュ TTL 24 時間、Netlify Functions で 6 時間ごとに
   定期更新、直近 7 日から成長率を計算。

```typescript
const maxChunkDays = 500; // 18 か月上限を安全に下回る
while (currentDate < today) {
  const chunkEnd = /* currentDate + 500 日、ただし today を超えない */;
  const url = `https://api.npmjs.org/downloads/range/${formatDate(currentDate)}:${formatDate(chunkEnd)}/${packageName}`;
  const data = await fetchWithRetry(url);
  chunks.push(data);
  currentDate = chunkEnd;
  await sleep(200); // レート制限回避
}
```

記事の結論は「単一の all-time リクエストを信用するな」「range + チャンクを使え」
「リトライを入れろ」「積極的にキャッシュしろ」「前提は実験で確かめろ」の 5 点。

#### 実装の構造

- クライアントは TanStack Query の `queryOptions` で、`queryKey` にパッケージ群と期間を
  含めてキャッシュを効かせる（`npmQueryOptions.ts`）。実データ取得はサーバ側の
  `fetchNpmDownloadsBulk` に集約し、**1 リクエストで複数パッケージをまとめて取る**。
- ビニングは d3 の UTC 単位（`d3.utcDay` / `utcWeek` / `utcMonth` / `utcYear`）で行い、
  起点を `2015-01-10`（npm のデータ開始日）に固定している（`binning.ts`）。
- 期間は 7 / 30 / 90 / 180 / 365 / 730 日などを用意。
- 表示は「実測」「ベースライン比較」「相対成長」を切り替えられ、`keepPreviousData` で
  期間切替時のちらつきを抑える。

#### 設計の工夫

- **ライブラリ単位の集約**: 1 リポジトリから出版された全パッケージ（スコープ付き、
  旧名、アドオン）を合算して「ライブラリの利用規模」を出す。`react-query` →
  `@tanstack/react-query` のような移行をまたいで追える。
- **前提を実験で検証している**。ドキュメントの記述を信じず、point と range の突き合わせで
  18 か月の切り詰めを発見した。この検証姿勢自体が最大の学び。
- 二重計上の可能性（アドオンがコアに依存する場合）を**README ではなくブログで明示**し、
  「今は単純化している」と限界を書いている。

#### 弱点

- npm 専用。他エコシステムへの一般化は未着手。
- 正確な全期間データを保つには数千回の API 呼び出しが必要で、更新コストが高い。
- ライブラリ集約は移行期のパッケージをまたぐため、定義によっては数値が膨らむ。

### npm-trend-clone（laststance/npm-trend-clone）

#### 何を作っているか

npmtrends.com のモダンなクローン。Next.js 16（App Router / Turbopack）、PostgreSQL（Neon）+
Prisma 7、Upstash Redis、Better Auth（GitHub OAuth）、TanStack Query、shadcn/ui という
構成で、**単なる可視化ではなくプロダクトとしての完成度**を狙っている。

#### データ取得

- `src/app/api/downloads/route.ts` が `packages`（カンマ区切り、最大 10 件）と
  `range`（`1m` / `3m` / `6m` / `1y` / `2y` / `5y`）を受け取り、パッケージごとに
  `api.npmjs.org/downloads/range/{start}:{end}/{pkg}` を **並列** に叩く。
- Next.js の fetch キャッシュを `next: { revalidate: 3600 }` で 1 時間に設定。
- 404 は「パッケージ未存在」として個別にエラーを返し、全体を失敗させない。

#### キャッシュとレート制限

- データキャッシュは Next.js の fetch キャッシュ（1 時間）。
- 自前 API の保護は `src/middleware.ts` の Upstash Ratelimit。**100 リクエスト / 10 秒 / IP** の
  スライディングウィンドウで、`X-RateLimit-*` ヘッダを返し、超過時は 429 + `Retry-After`。
- Redis 障害時は **fail-open**（通す）。監視用の `/api/health` はレート制限の対象外。

```typescript
const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(100, "10 s"),
  ephemeralCache: new Map(),
  analytics: true,
  prefix: "npm-trend-ratelimit",
});
```

#### 指標の見せ方

- 複数パッケージの比較チャート、パッケージ情報カード、カテゴリ別プリセット
  （`src/constants/category-presets.ts`）、プリセットの保存（DB + 認証）。
- 共有ボタン、エクスポート、テーマ切替、オフライン表示、検索バー。
- ストーリー（`*.stories.tsx`）とテスト（Vitest / Playwright / MSW）が揃っており、
  コンポーネント単位の検証体制がある。

#### 設計の工夫

- **npm API を自前の API route で包む**ことで、フロントエンドが外部 API の仕様変更に
  直接さらされない。レート制限・エラー処理・キャッシュを一箇所に集約できる。
- 認証・プリセット・共有まで含む「サービスとしての形」を持ち、プロダクト設計の参考になる。
- E2E で MSW を使い、外部 API に依存しないテストを実現している。

#### 弱点

- **期間指定が 18 か月上限を考慮していない**。`2y` / `5y` は `start:end` をそのまま
  npm API に渡すため、TanStack の検証によれば実際には約 549 日分しか返らない。
  画面は 5 年分を表示しているつもりでも、データは 1 年半程度しかない。
- npm 専用。
- 認証・DB・Redis が必須で、セルフホストの敷居が高い（ローカル開発に Docker 3 サービス）。

## 比較

### 観点別の比較表

| 観点 | npm.bet | TanStack NPM Stats | npm-trend-clone |
| --- | --- | --- | --- |
| 対象 | npm のみ | npm のみ（TanStack の 200+ パッケージ） | npm のみ |
| データ取得 | server action から直接 | サーバ関数に集約し一括取得 | 自前 API route 経由で並列取得 |
| 全期間の扱い | 単一リクエスト（18 か月で切詰め） | 500 日チャンク + 合算 + リトライ | 単一リクエスト（同上） |
| キャッシュ | Upstash Redis 1 時間（`after()` で非同期保存） | DB 24 時間 + 6 時間ごとの定期更新 | Next fetch キャッシュ 1 時間 |
| レート制限 | なし（キャッシュのみ） | 取得側で 200ms 間隔・8 並列 | 自前 API に 100 req/10 秒/IP |
| 欠測の扱い | ゼロ補間をオプトイン（実測と推定を分離） | 実測のみ（ゼロはゼロとして表示） | 特段の扱いなし |
| 集約 | 日／週／月 | 日／週／月／年（UTC 固定） | 期間プリセットのみ |
| 出力 | PNG / SVG 埋め込み / OG | SVG 埋め込み / 共有 URL | 共有 / エクスポート |
| 認証 | なし | なし（自社サイトの一部） | GitHub OAuth + DB |
| テスト | 純粋関数の単体テスト + フィクスチャ | 型と計算ロジックの分離 | Vitest / Playwright / MSW |
| 強み | 可視化と埋め込みの完成度 | 正確性への執念と設計の言語化 | プロダクトとしての網羅性 |

### 3 実装に共通する弱点

1. **すべて npm 専用**。言語横断の抽象（PURL、エコシステム差異、ダウンロード数を持たない
   レジストリ）は誰も解いていない。ここが libtrends の空白地帯。
2. **健全性・脆弱性を扱っていない**。3 実装とも「ダウンロード数の推移」が主目的で、
   EOL・保守性・脆弱性は指標に含まれない。
3. **データの意味の説明が弱い**。ダウンロード数が CI やミラーを含むこと、パッケージ名の
   移行で系列が切れることは、TanStack が FAQ で触れている程度。

### libtrends への示唆

1. **取得はサーバ側に集約し、クライアントは期間とパッケージ群をキーにキャッシュする**
   （TanStack 方式）。フロントが外部 API を直接叩くと、レート制限と仕様変更の影響を
   もろに受ける。
2. **履歴はチャンクで取り、取れた範囲を画面に明示する**。18 か月の切り詰めは npm だけで
   起きる問題ではない。どの API でも「要求した期間」と「返ってきた期間」を突き合わせ、
   乖離があれば `meta` に記録して UI に出す（04-design-draft.md の `meta.missing` に対応）。
3. **実測値と推定値を別フィールドで持つ**（npm.bet 方式）。補間や補完をしても原データを
   失わない構造は、後から検証するときに効く。
4. **キャッシュ TTL はデータの更新頻度に合わせて層別にする**。ダウンロードは 1 時間〜1 日、
   メタデータは日次、健全性は週次。一律 TTL は無駄か鮮度不足のどちらかになる。
5. **レート制限は「外向き（取得）」と「内向き（公開 API）」の両方に要る**。
   取得側はチャンク間の待機と並列度制御、公開側は IP 単位のスライディングウィンドウ
   （npm-trend-clone 方式）が実装しやすい。
6. **前提を実験で確かめる**。TanStack の貢献は実装そのものより「API の挙動を検証して
   記事にした」点が大きい。libtrends でも各 API の上限・切り詰め・欠測を実測で確かめ、
   資料に残す価値がある。

## 出典

- npm.bet リポジトリ: https://github.com/vercel/npm.bet
- npm.bet 同梱 README / `actions/package/get.ts` / `lib/download-data.ts`:
  `docs/research/references/npm-bet.md`
- TanStack 設計ブログ（"How We Track Billions of Downloads"、2025-12-02）:
  https://tanstack.com/blog/npm-stats-the-right-way
- TanStack サイトのリポジトリ: https://github.com/TanStack/tanstack.com
- TanStack 実装（`src/components/npm-stats/`、`src/blog/npm-stats-the-right-way.md`）:
  `docs/research/references/tanstack-npm-stats.md`
- npm-trend-clone リポジトリ: https://github.com/laststance/npm-trend-clone
- npm-trend-clone 実装（`src/app/api/downloads/route.ts`、`src/middleware.ts`）:
  `docs/research/references/npm-trend-clone.md`
- npm downloads API 公式ドキュメント: https://github.com/npm/registry/blob/master/docs/download-counts.md
