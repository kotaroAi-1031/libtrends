# libtrends

🌐 [English](README.md)

**ライブラリの指標を、あらゆるレジストリ横断で。**

libtrends は、ソフトウェアライブラリを表す指標——ダウンロード数、
スター数、人気の推移、依存関係、保守の健全性、既知の脆弱性——を、
言語とパッケージレジストリを横断して1か所に集積するプロジェクトです。

「npm trends」の npm 以外への拡張、とも言えます。npm、PyPI、crates.io、
RubyGems、Packagist、Maven Central、Go モジュールなどについて、
同じ指標を共通のモデルに正規化して並べます。

## 扱う指標

| 指標の系統 | 例 |
| --- | --- |
| 人気 | ダウンロード数と推移、GitHub スター数、被依存数 |
| 活動 | リリース間隔、最終リリース、コミット活動 |
| 健全性 | EOL・放棄の状態（uzomuzo の信号）、メンテナの応答性 |
| セキュリティ | 既知の脆弱性（vuls と OSV のデータ） |

## 現在の状態

🚧 **調査フェーズ。** 設計を固定する前に、利用できる集計 API と
既存の実装を調査しています。

- 調査資料: [`docs/research/`](docs/research/README.md)
  （API の網羅調査、既存実装の分析、健全性信号の設計、設計たたき台）
- リファレンス実装は [Repomix](https://github.com/yamadashy/repomix) で
  パックして [`docs/research/references/`](docs/research/references/) に
  置いています——
  [vercel-labs/npm.bet](https://github.com/vercel-labs/npm.bet)、
  [TanStack/tanstack.com](https://github.com/TanStack/tanstack.com) 内の
  npm stats ツール、npm trends のモダンなクローンの3点です。
- 調査資料は構造化 Markdown で書き、整合性の維持には
  [contextlint](https://github.com/nozomi-koborinai/contextlint) を
  使います（`contextlint.config.json` を参照）。

## データソース（予定）

- npm downloads API / registry API
- [libraries.io](https://libraries.io/api)
- [deps.dev](https://deps.dev/) と OSV
- [ecosyste.ms](https://ecosyste.ms/)
- GitHub API
- レジストリごとの API（PyPI、crates.io、RubyGems、Packagist、Maven Central、Go module proxy）
- 健全性信号としての uzomuzo（EOL・放棄の検出）と vuls（脆弱性スキャン）の出力

## リポジトリの規約

このリポジトリは `muse-repo-template` テンプレートから作られました。
Lefthook による Betterleaks のシークレット検査、堅牢化した CI リント、
ツール版管理の正としての Nix flake（`flake.nix` + `flake.lock`）が
最初から入っています。変更はプルリクエスト経由で入れ、`main` へは
直接プッシュしません。
