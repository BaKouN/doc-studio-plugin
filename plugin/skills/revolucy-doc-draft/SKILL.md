---
name: revolucy-doc-draft
description: Rédige un livrable client Revolucy (audit, arborescence, etc. — types industrialisés exposés par `list_doc_types`) à partir d'un brief utilisateur. Les règles de rédaction (ton, vocabulaire, contraintes) sont chargées au runtime depuis les MCP resources `revolucy://context/*`. Push le brouillon JSON au studio via le tool MCP `create_doc` et retourne l'URL d'édition. Utiliser quand l'utilisateur a un brief prospect (notes meeting, audit technique, URL avec observations) et veut un livrable charté avec form web pour ajuster.
---

# Revolucy Doc Draft

Tu rédiges un brouillon JSON conforme au schéma d'un type de document supporté par le studio Revolucy, tu le push via le tool MCP `create_doc`, et tu retournes l'URL d'édition à l'utilisateur. Il ajustera dans le form web puis exportera le PDF.

## Serveur MCP

Toutes les resources `revolucy://...` et les tools sont servis par un seul serveur MCP HTTP distant. Quand tu appelles `readMcpResource`, le paramètre `server` doit être exactement :

```
plugin:revolucy-doc-studio:revolucy-studio
```

(Nom namespaced injecté par Claude Code pour les MCP servers fournis par un plugin.)

L'auth se fait en OAuth 2.1 — Claude Code ouvre une fenêtre navigateur au premier 401. Si tu n'as jamais été whitelisté côté studio, l'admin Revolucy doit ajouter ton email avant.

## Pré-flight : charger le contexte Revolucy

**Avant toute rédaction**, lis ces resources MCP. Elles portent les règles de rédaction, le positionnement, et les do/don't. Tu ne dois rien drafter sans les avoir au minimum lues :

- `revolucy://context/ton-redactionnel` — règles de ton, vocabulaire, contraintes typographiques
- `revolucy://context/positionnement` — qui est Revolucy, à qui on s'adresse
- `revolucy://context/do-dont` — interdits absolus

Si le draft touche un sujet précis, charge aussi la resource correspondante :

- `revolucy://context/avantages-concurrentiels` — si le draft mentionne un avantage concurrentiel
- `revolucy://context/expertises` — si le draft parle d'une expertise technique
- `revolucy://context/process` — si le draft décrit le process projet
- `revolucy://context/clients-passes` — si le draft cite des références

## Pré-flight : charger le schéma + l'exemple du doc_type

Pour chaque `doc_type` :

- `revolucy://schema/<doc_type>` — schéma YAML qui décrit chaque champ avec sa contrainte sémantique
- `revolucy://example/<doc_type>` — exemple JSON déjà rempli (ancrage de qualité, à imiter en structure, pas en contenu)

## Workflow

1. **Vérifier l'accès au studio.** Appelle `list_doc_types`. Si l'appel échoue avec 401, l'auth OAuth n'a pas été négociée ou le compte n'est pas whitelisté. Demande à l'utilisateur de relancer l'install ou de contacter l'admin.

2. **Confirmer le `doc_type`.** Si l'utilisateur n'a pas précisé, présente la liste obtenue via `list_doc_types`. Si le type demandé est absent, appelle `validate_doc_type(<type>)` pour distinguer "type jamais industrialisé" vs "type en cours". Redirige vers le skill `revolucy-template-creator` au besoin.

3. **Charger les resources** (cf. pré-flight).

4. **Évaluer la couverture du brief.** Le contexte fourni est-il suffisant pour drafter sans inventer ? Si non, demande des précisions ciblées avant de drafter. Ne JAMAIS inventer une métrique, un constat, une URL, un chiffre.

5. **Drafter le JSON** conforme au schéma chargé. Respecte strictement les règles chargées via `revolucy://context/ton-redactionnel` et `revolucy://context/do-dont`. Si une règle n'est pas claire, charge la resource correspondante avant de drafter.

6. **Push via `create_doc`.** Récupère le `edit_url` retourné.

7. **Retourner à l'utilisateur** :
   - L'URL d'édition (`edit_url`)
   - 2-3 zones du draft à vérifier en priorité (champs où tu as comblé une zone ambiguë du contexte)
   - Optionnel : ce qui manquait dans le brief si pertinent pour le prochain coup

## Règles d'or

### 1. Zéro contenu inventé

Tu rédiges UNIQUEMENT à partir :
- du contexte explicitement fourni par l'utilisateur (transcript, audit, URL avec observations)
- du contexte Revolucy chargé via les resources MCP `revolucy://context/*`
- de faits observables et vérifiables dans le contexte fourni

**Interdit** : inventer une métrique, paraphraser un USP qui n'a jamais été énoncé, gonfler un chiffre. Si tu n'es pas sûr d'un fait, rends le champ plus court ou plus générique plutôt que faux.

Le doc va devant un client BtoB qui vérifie. Une donnée inventée = perte de crédibilité.

### 2. Respect strict des règles chargées

Les règles de ton, vocabulaire et typographie vivent dans `revolucy://context/ton-redactionnel`. **Avant de pousser un draft**, relis tes champs et confronte-les à ces règles. Tu n'écris **rien** qui contredise une règle chargée.

### 3. Auto-vérification avant push

Avant d'appeler `create_doc`, fais une passe :

- Toutes les règles de `revolucy://context/ton-redactionnel` respectées
- Aucun interdit de `revolucy://context/do-dont` enfreint
- Aucune métrique non sourcée dans le contexte fourni
- Tous les `required: true` du schéma renseignés
- **Tous les `max_words` du schéma respectés** (cf. règle 4)

### 4. Respecter les `max_words` du schéma — self-count avant push

Le schéma déclare un `max_words` par champ texte (paragraphes, titres, items d'array). Ces caps sont calibrés pour que le contenu **tienne sur A4** une fois rendu PDF — les dépasser = contenu clippé en PDF même si la preview avait l'air OK.

**Règle dure** : avant `create_doc`, pour chaque champ texte de ton draft, compte les mots après strip des tags HTML (`<em>`, `<strong>`, `<span>`, etc. ne comptent pas comme mots). Si tu dépasses un `max_words`, **trim avant d'appeler** — ne push pas un draft qui sera refusé.

Exemple : si `intro_paragraphe.max_words: 150` et tu as drafté 180 mots, soit tu raccourcis dans ton draft, soit tu coupes la phrase la moins essentielle avant push. Pas de generation/refus loop.

Le studio enforce comme garde-fou côté serveur : si tu push quand même un champ trop long, tu reçois :

```json
{
  "error": "fields_too_long",
  "doc_type": "audit",
  "violations": [
    {"field": "intro_paragraphe", "max_words": 150, "actual_words": 180},
    {"field": "constats[2].corps", "max_words": 110, "actual_words": 142}
  ]
}
```

Dans ce cas : retrim les champs listés (avec le delta `actual - max` à enlever), re-push. Mais l'objectif est zéro roundtrip — self-check d'abord.

**Note sur le comptage** : "mots" = tokens séparés par whitespace, **après strip des tags HTML**. `<em>2 leads qualifiés</em>` = 3 mots. La ponctuation collée ne crée pas de mot supplémentaire (`hello,` = 1 mot).

## Après push

Format de sortie recommandé :

> Brouillon poussé. URL d'édition : `<edit_url>`
>
> À vérifier en priorité dans le form :
> - `<champ>` : `<raison>`
> - `<champ>` : `<raison>`
>
> Quand c'est validé, cliquez "Exporter PDF" en bas du form.

## Si le type demandé n'est pas supporté

Si `list_doc_types` ne retourne pas le type demandé :

> Le type `<demandé>` n'est pas encore supporté par le studio. Types actuels : `<liste>`. Pour l'industrialiser, lance le skill `revolucy-template-creator` (HTML chartet de référence requis).

Si `validate_doc_type(<demandé>)` retourne `complete: false, missing: [partiel]`, dis-le : « Le type `<demandé>` est en cours d'industrialisation (manque : `<missing>`). Relance `revolucy-template-creator` pour terminer ou rollback. »
