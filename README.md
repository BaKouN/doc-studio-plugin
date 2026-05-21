# Revolucy Doc Studio — plugin Claude

Plugin Claude qui connecte le studio Revolucy (`doc-studio.revolucydev.com`) en MCP HTTP. Fournit les tools de génération de livrables (audit, arborescence, dossier de conception) et les skills associés. Auth OAuth 2.1 (Google), accès réservé aux utilisateurs whitelistés Revolucy.

## Install

### Sur Claude Code CLI

```
/plugin marketplace add BaKouN/doc-studio-plugin
/plugin install revolucy-doc-studio@revolucy
```

Puis quitte et relance Claude.

### Sur Cowork desktop

Le `/plugin` n'existe pas sur Cowork — passer par l'UI :

**Paramètres → Connecteurs → + → Ajouter des connecteurs personnalisés**

- Nom : `revolucy-studio`
- URL : `https://doc-studio.revolucydev.com/mcp`

Cliquer Ajouter → login Google Revolucy dans le browser.

---

Guide complet (premier prompt, troubleshooting, etc.) : **[ONBOARDING.md](ONBOARDING.md)**.
