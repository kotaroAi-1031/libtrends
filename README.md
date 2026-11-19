# libtrends

🌐 [日本語](README.ja.md)

**Library metrics, across every registry.**

libtrends aggregates the metrics that describe a software library —
download counts, stars, popularity trends, dependency relationships,
maintenance health, and known vulnerabilities — across languages and
package registries, in one place.

Think "npm trends", but not just for npm: the same library indicators,
normalized into a common model for npm, PyPI, crates.io, RubyGems,
Packagist, Maven Central, Go modules, and more.

## What it tracks

| Metric family | Examples |
| --- | --- |
| Popularity | Download counts and trends, GitHub stars, dependents |
| Activity | Release cadence, last release, commit activity |
| Health | End-of-life / abandoned status (uzomuzo signals), maintainer responsiveness |
| Security | Known vulnerabilities (via vuls and OSV data) |

## Status

🚧 **Research phase.** The project is currently surveying the available
aggregation APIs and existing implementations before the design is fixed.

- Research documents: [`docs/research/`](docs/research/README.md)
  (API survey, analysis of existing implementations, health-signal
  design, design draft)
- Reference implementations packed with
  [Repomix](https://github.com/yamadashy/repomix) live in
  [`docs/research/references/`](docs/research/references/) —
  [vercel-labs/npm.bet](https://github.com/vercel-labs/npm.bet),
  the npm stats tool inside
  [TanStack/tanstack.com](https://github.com/TanStack/tanstack.com),
  and a modern npm-trends clone.
- The research documents are structured Markdown, kept consistent with
  [contextlint](https://github.com/nozomi-koborinai/contextlint)
  (see `contextlint.config.json`).

## Data sources (planned)

- npm downloads / registry APIs
- [libraries.io](https://libraries.io/api)
- [deps.dev](https://deps.dev/) and OSV
- [ecosyste.ms](https://ecosyste.ms/)
- GitHub API
- Per-registry APIs (PyPI, crates.io, RubyGems, Packagist, Maven Central, Go module proxy)
- uzomuzo (EOL / abandonment detection) and vuls (vulnerability scanning) outputs as health signals

## Repository conventions

This repository was created from the `muse-repo-template` template:
Betterleaks secret scanning via Lefthook, hardened CI lint, and a Nix
flake toolchain (`flake.nix` + `flake.lock`) as the source of truth for
tool versions. Changes land through pull requests; `main` is never
pushed to directly.
