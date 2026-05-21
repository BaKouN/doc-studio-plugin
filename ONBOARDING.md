# Revolucy Doc Studio — Démarrer en 5 minutes

> Guide d'installation pour les utilisateur·rices du studio.
>
> Deux clients possibles : **Claude Code CLI** (terminal) ou **Cowork desktop** (l'app Anthropic). Suis la section qui correspond à ton outil.

---

## Pré-requis (pour les deux)

1. **Une app Claude installée**, soit :
   - **Claude Code** (CLI Mac/Windows, dans un terminal), OU
   - **Cowork desktop** (app Mac/Windows par Anthropic).
2. **Ton email Revolucy whitelisté côté studio.** Si tu n'es pas sûr·e, demande à un admin du studio.
3. **Un navigateur connecté à ton compte Google Revolucy** (Chrome, Safari, peu importe). Le studio utilise Google pour l'auth.

Tu n'as **pas besoin** de Python, git, Docker, ni de cloner quoi que ce soit. Tout tourne sur le serveur ; toi tu connectes Claude au serveur, c'est tout.

---

## A. Si tu utilises Claude Code CLI

Dans Claude Code, **dans la fenêtre de chat** (peu importe le dossier où tu es) :

```
/plugin marketplace add BaKouN/doc-studio-plugin
/plugin install revolucy-doc-studio@revolucy
```

Claude charge le plugin en 3-5 secondes et te confirme `✔ Successfully installed plugin: revolucy-doc-studio@revolucy`.

**Quitte et relance Claude** (tape `exit` puis relance `claude`).

> ℹ️ Pour vérifier que le plugin est bien actif après restart : tape `/plugin` — tu dois voir `revolucy-doc-studio@revolucy ✔ enabled` avec `1 MCP server (revolucy-studio)` et `2 skills`.

Saute maintenant à **[3. Première utilisation](#3-première-utilisation)** plus bas.

---

## B. Si tu utilises Cowork desktop

Cowork desktop n'a **pas** la commande `/plugin` (c'est spécifique à Claude Code CLI). Tu passes par l'UI Connecteurs :

1. Ouvre Cowork desktop.
2. Va dans **Paramètres** (icône ⚙️) → **Connecteurs** → bouton **+** → **Ajouter des connecteurs personnalisés**.
3. Remplis :
   - **Nom** : `revolucy-studio` (ou ce que tu veux)
   - **URL** : `https://doc-studio.revolucydev.com/mcp`
4. Clique **Ajouter**. Une fenêtre browser s'ouvre pour le login Google → valide.

Cowork connecte le serveur MCP via l'infra cloud Anthropic. Pas besoin de restart.

> ⚠️ Limitation Cowork desktop : tu n'as **pas** les "skills" qui auto-triggerent sur "drafte-moi un audit" (elles sont spécifiques à Claude Code CLI). Tu dois être un peu plus explicite dans ton premier prompt — voir plus bas.

---

## 3. Première utilisation

### Sur Claude Code CLI

Demande directement, en langage naturel, ce que tu veux :

```
Drafte-moi un audit pour le prospect <à toi de remplir>, à partir
de ces notes :
URL : ...
Secteur / activité : ...
Observations clés :
- ...
- ...
```

Les skills détectent que tu veux drafter un audit et appellent les bons tools studio. Tu reçois un lien `https://doc-studio.revolucydev.com/docs/XXXXXXX`.

### Sur Cowork desktop

Comme tu n'as pas les skills, ton premier prompt de la session doit demander explicitement à Claude de charger le contexte avant de drafter :

```
Utilise le connecteur revolucy-studio. Liste d'abord ses resources MCP
et lis celles qui ressemblent à du contexte (positionnement, ton, etc.)
avant de drafter quoi que ce soit. Ensuite, drafte-moi un audit pour le
prospect <nom>, à partir de ces notes :
URL : ...
Secteur / activité : ...
Observations clés :
- ...
- ...
```

Une fois que Claude a chargé le contexte dans la session, tu peux enchaîner les drafts sans relire les resources.

### Dans les deux cas

Au tout premier appel, Claude négocie l'auth via le browser :

1. **Si tu n'es pas connecté à Google** : sélectionne ton compte Revolucy, accepte les permissions.
2. **Si tu es déjà connecté** : page blanche style _"vous êtes connecté(e)"_ qui se ferme toute seule.
3. **Retour à Claude** : le draft se génère, et tu reçois un lien `https://doc-studio.revolucydev.com/docs/XXXXXXX`.

Ouvre ce lien dans ton browser (tu es déjà loggé côté studio), ajuste 1-2 détails dans le formulaire, clique **Exporter PDF** → le PDF charté se télécharge dans `Téléchargements`.

**C'est tout.** Une fois fait, tu peux fermer/relancer Claude autant que tu veux — le token reste mémorisé.

---

## En cas de souci

| Symptôme | Que faire |
|---|---|
| Le plugin / connecteur n'apparaît pas après install | Quitte complètement Claude (Cmd+Q sur Mac) puis relance. |
| Erreur 403 / _"Compte non autorisé"_ au login Google | Ton email n'est pas whitelisté. Demande à un admin. |
| Le navigateur ne s'ouvre pas tout seul | Claude affiche l'URL dans le chat (commence par `https://doc-studio.revolucydev.com/authorize`). Copie-colle dans Chrome manuellement. |
| Erreur 401 après usage régulier | Ton token a expiré (1h). Re-déclenche un tool, Claude refait l'auth automatiquement. |
| Claude écrit un script Python au lieu d'utiliser le studio | Le MCP server n'est pas connecté. Re-vérifie l'install (section A ou B). |

> Le studio est un **outil interne best-effort** : pas de support garanti. Pour tout autre problème, garde un screenshot dans Slack et attends qu'on regarde.

---

## Mise à jour plus tard (CC CLI seulement)

```
/plugin update revolucy-doc-studio@revolucy
```

Claude refetch depuis GitHub et reload le plugin. Pas besoin de re-login.

Sur Cowork desktop : pas de mise à jour côté client à faire, c'est le serveur qui héberge tout.

---

## Architecture (si tu es curieux)

- **Le plugin** : 2 skills + 1 fichier de config qui pointe vers le serveur MCP HTTP. Sources publiques : https://github.com/BaKouN/doc-studio-plugin
- **Le serveur MCP** : tourne sur `https://doc-studio.revolucydev.com/mcp`, expose des tools (drafter, lister, exporter PDF) et des resources de contexte auth-gated.
- **Auth** : OAuth 2.1 standard (PKCE), négocié automatiquement au 1er appel. Aucun token API à manipuler à la main.
- **Whitelist** : liste d'emails côté serveur, gérée par les admins.
