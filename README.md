# Revolucy Doc Studio — plugin Claude Code

Plugin Claude Code qui connecte le studio Revolucy (`doc-studio.revolucydev.com`) en MCP HTTP. Fournit les tools de génération de livrables (audit, arborescence, dossier de conception) et les skills associés. Auth OAuth 2.1 (Google), accès réservé aux utilisateurs whitelistés.

## Install

```
/plugin marketplace add BaKouN/doc-studio-plugin
/plugin install revolucy-doc-studio@revolucy
```

Au premier appel d'un tool, Claude Code ouvre une fenêtre navigateur pour le login Google. Si votre compte n'est pas whitelisté, contactez l'admin Revolucy.
