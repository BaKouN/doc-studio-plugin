# Changelog

Toutes les évolutions notables du plugin Revolucy Doc Studio. Format inspiré de
[Keep a Changelog](https://keepachangelog.com/). Une entrée par version
publiée (bump de `plugin/.claude-plugin/plugin.json` + tag).

Catégories : `Added`, `Changed`, `Fixed`, `Removed`, `Security`.

## [Unreleased]

## [0.2.3] — 2026-05-27

### Changed

- Marketplace migré de `github.com/BaKouN/doc-studio-plugin` à
  `gitlab.com/agence_revolucy/doc-studio-plugin`. Branding agence Revolucy
  unifié. Les utilisateurs existants doivent retirer l'ancien marketplace
  et ajouter le nouveau :
  ```
  /plugin marketplace remove revolucy
  /plugin marketplace add https://gitlab.com/agence_revolucy/doc-studio-plugin.git
  /plugin install revolucy-doc-studio@revolucy
  ```
- `README.md` et `ONBOARDING.md` mis à jour avec la nouvelle URL marketplace.

## [0.2.2] — 2026-05-22

### Changed

- Bump version pour aligner le marketplace et le plugin sur l'état stable
  post-déploiement studio prod (HTTP MCP + OAuth 2.1 live, validation
  end-to-end avec 4 users internes).

## [0.2.1] — antérieur

### Added

- Marketplace `.claude-plugin/marketplace.json` au root du repo pour permettre
  `/plugin marketplace add` + `/plugin install revolucy-doc-studio@revolucy`.
- Skills `revolucy-doc-draft` et `revolucy-template-creator`.
- MCP server config HTTP+OAuth pointant sur `https://doc-studio.revolucydev.com/mcp`.

[Unreleased]: https://gitlab.com/agence_revolucy/doc-studio-plugin/-/compare/v0.2.3...HEAD
[0.2.3]: https://gitlab.com/agence_revolucy/doc-studio-plugin/-/tags/v0.2.3
[0.2.2]: https://gitlab.com/agence_revolucy/doc-studio-plugin/-/tags/v0.2.2
[0.2.1]: https://gitlab.com/agence_revolucy/doc-studio-plugin/-/tags/v0.2.1
