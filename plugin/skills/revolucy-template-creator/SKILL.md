---
name: revolucy-template-creator
description: Industrialise un nouveau type de livrable Revolucy (template Jinja2 + schéma YAML + example JSON, form optionnel) à partir d'un HTML chartet de référence. Itère en chat via le tool MCP `preview_user_template`, persiste via `submit_user_template` (espace perso) ou `submit_global_template` (admin). Utiliser pour ajouter un livrable récurrent (devis, dossier de conception, PV de signature, etc.) au catalogue du studio.
---

# Revolucy Template Creator

Tu industrialises un nouveau type de livrable Revolucy. À partir d'un HTML chartet de référence fourni par l'utilisateur, tu produis les artifacts qui permettent au studio ET au skill `revolucy-doc-draft` de manipuler ce nouveau type :

1. `template.html` (Jinja2 — rendu PDF)
2. `schema.yaml` (schéma sémantique)
3. `example-default.json` (ancrage de qualité, rendu testé)
4. `form.html` (optionnel — uniquement si UI custom nécessaire)

Tu n'écris **aucun fichier** côté studio. Tu valides en chat via le tool MCP `preview_user_template` (rendu HTML + erreurs structurées), puis tu persistes via `submit_user_template` (espace perso) ou `submit_global_template` (admin uniquement). Le studio s'occupe d'écrire au bon endroit.

## Quand l'utiliser

Triggers explicites :
- L'utilisateur dit « industrialiser un nouveau type », « créer un template pour <doc> », « ajouter un livrable au studio »
- L'utilisateur a un HTML chartet de référence et veut le rendre re-templatable
- L'utilisateur veut compléter un type partiellement créé (`validate_doc_type` retourne `complete: false, missing: [partiel]`)

Ne PAS utiliser :
- Pour drafter un livrable sur un type existant → `revolucy-doc-draft`
- Pour un PDF technique à partir d'un markdown → `revolucy-md-to-pdf`
- Si l'utilisateur n'a pas de HTML chartet de référence — refuser poliment

## Serveur MCP

Toutes les resources `revolucy://...` et les tools sont servis par un seul serveur MCP HTTP distant. Quand tu appelles `readMcpResource`, le paramètre `server` doit être exactement :

```
plugin:revolucy-doc-studio:revolucy-studio
```

L'auth se fait en OAuth 2.1 — Claude Code ouvre une fenêtre navigateur au premier 401. Si tu n'as jamais été whitelisté côté studio, l'admin doit ajouter ton email avant.

## Pré-flight : charger le contexte Revolucy

Au minimum :

- `revolucy://context/ton-redactionnel`
- `revolucy://context/positionnement`
- `revolucy://context/do-dont`
- `revolucy://context/charte-graphique`

Et selon le type demandé :

- `revolucy://context/process` — types liés à la gestion projet
- `revolucy://context/expertises` — types techniques (dossier de conception, audit)
- `revolucy://context/avantages-concurrentiels` — types commerciaux (devis, plaquette, proposition)
- `revolucy://context/clients-passes` — types citant des références

## Workflow

### 1. Comprendre la demande + slug check

Demande à l'utilisateur :

- Nom du nouveau type (slug kebab-case court — `devis`, `dossier-conception`, etc.)
- Le HTML chartet de référence (collé en chat, ou contenu d'un fichier qu'il colle)
- À qui le doc est destiné, dans quel moment du process
- Mode `--global` (admin) ou flux standard utilisateur ?

**Slug check** : appelle `validate_doc_type(<slug>)`. Trois cas :

- `complete: true` → **collision**. Refuse : « Le type `<slug>` existe déjà et il est complet. Pour l'écraser, retry avec `submit_global_template(overwrite=true)` (admin uniquement). Sinon, choisis un autre slug. »
- `complete: false, missing: [tout]` → fresh start, continue.
- `complete: false, missing: [partiel]` → reprise interrompue. Propose à l'utilisateur de redresser : repartir des artifacts existants (lis-les via les resources `revolucy://schema/<slug>` etc.) ou repartir from scratch.

Si l'utilisateur n'a pas de HTML de référence, refuse poliment : « Pour industrialiser un type, j'ai besoin d'une maquette chartée existante. »

### 2. Lire le HTML de référence

Identifie :

- Les zones de contenu variable (texte client, listes, blocs répétables)
- Les zones décoratives fixes
- Les blocs réutilisables Revolucy déjà décrits dans `revolucy://context/charte-graphique`
- L'orientation et le nombre de pages

Annonce ton analyse en chat avant de proposer le schema.

### 3. Auditer le knowledge base Revolucy

Identifie ce qui manque dans les resources `revolucy://context/*` pour drafter ce type sans réexplication future. Note-le et propose à l'utilisateur de compléter (édition manuelle hors skill).

### 4. Proposer le `schema.yaml`

Modèle de référence : `revolucy://schema/audit` (lis-le avant de proposer ton schéma).

Règles :

- `required: true` pour les champs sans lesquels le draft est invalide (le studio rejettera tout draft qui les omet avec 400 `missing_required`). N'en abuse pas.
- Décris chaque champ par son intention sémantique, pas juste son type.
- Mappe les tags HTML autorisés au rendu attendu si le template les utilise.

Propose le schéma intégralement dans le chat. Attends validation.

### 5. Proposer le `template.html`

Conversion HTML chartet → Jinja2 :

- Variables `{{ data.<champ> }}` (avec `| safe` si tags HTML autorisés)
- Boucles `{% for item in data.<array> %}...{% endfor %}`
- Garde `<head>` (fonts, CSS print) tel quel
- Bloc preview `@media screen` (overflow A4) si pertinent

Contrat dur : pages dans `<section class="page...">` non-imbriquées. Le renderer split par regex sur cette balise.

Le renderer utilise `StrictUndefined` — chaque variable du template doit avoir une valeur dans `data`, ou être protégée par `{% if data.foo %}`. Toute variable manquante remonte en `template_undefined` au preview.

**Règle dure** : tu ne hardcodes JAMAIS de données prospect dans le template. Le template est abstrait, l'example porte la donnée concrète.

Propose le template intégralement dans le chat. Attends validation.

### 6. Proposer le `example-default.json`

Basé sur un cas réel (préféré) ou inventé conforme à la charte. C'est l'ancrage de qualité du type — il sert aussi à drafter le rendu test au moment du preview.

Le studio stocke un seul example par type sous `example-default.json`.

### 7. Valider le bundle via `preview_user_template`

Appelle :

```
preview_user_template(
  doc_type=<slug>,
  template_html=<le template proposé>,
  schema_yaml=<le YAML proposé>,
  example_json=<le JSON proposé, en dict>,
  form_html=<optionnel, seulement si UI custom>,
)
```

Le tool retourne **soit** `{ok: true, html: <HTML rendu>}` (succès), **soit** un `ToolError` portant un `detail` structuré (400). Tu interprètes chaque shape :

- **`{error: "incomplete", missing: [...]}`** — un artifact requis (template/schema/example) manque ou est vide.
- **`{error: "invalid_schema", message: "..."}`** — `schema.yaml` n'est pas un YAML valide ou pas un mapping. Corrige la syntaxe.
- **`{error: "schema_template_drift", missing_in_template: [...], extras_in_template: [...]}`** — désynchro schema/template. Aligne les deux.
- **`{error: "template_undefined", message, ...}`** — le template référence `data.<x>` qui n'existe pas dans l'example. Soit ajouter le champ dans l'example, soit protéger par `{% if data.<x> %}`.
- **`{error: "template_syntax", message, line}`** — erreur de syntaxe Jinja2. Corrige.
- **`{error: "render_failed", message, ...}`** — autre erreur au rendu. Lis le message.

En succès : présente le HTML rendu pour validation visuelle. Sauve dans un fichier temp (`/tmp/preview-<slug>.html`) et donne le chemin pour que l'utilisateur l'ouvre dans son navigateur. Demande son OK avant de soumettre.

Tu peux re-preview autant de fois que nécessaire — `preview_user_template` ne persiste rien.

### 8. Soumettre

#### Flux standard (utilisateur)

```
submit_user_template(
  doc_type=<slug>,
  template_html=...,
  schema_yaml=...,
  example_json=...,
  form_html=<optionnel>,
)
```

Le studio écrit dans `data/user_templates/<user_id>/<doc_type>/` et retourne `{ok, doc_type, user_id, paths: [...]}`. Idempotent dans l'espace perso — re-submit écrase.

Annonce :

> Ton template `<slug>` est créé dans ton espace perso (`<paths>`). Tu peux le drafter via `revolucy-doc-draft`. Pour publication globale, soumets-le à l'admin via `/admin/templates`.

#### Flux admin (`--global`)

Si l'utilisateur est admin et a explicitement demandé la publication directe :

```
submit_global_template(
  doc_type=<slug>,
  template_html=...,
  schema_yaml=...,
  example_json=...,
  form_html=<optionnel>,
  overwrite=False,
)
```

Si non-admin : 403 (`ToolError`). Dis-le sans détour : « Tu n'es pas admin, je ne peux pas publier globalement. Soumis dans ton espace perso à la place. »

Si l'admin appelle sur un slug existant : 409 `{error: "target_exists", existing: [chemins]}`. Présente la liste, demande s'il veut écraser, retry avec `overwrite=True`.

### 9. Sortie attendue (succès)

> Type `<doc_type>` industrialisé.
>
> Bundle soumis :
> - template (`<chemin>`)
> - schema (`<chemin>`)
> - example (`<chemin>`)
> - form (`<chemin>` si custom)
>
> Le rendu a passé le preview (HTML test ok). Le skill `revolucy-doc-draft` peut maintenant drafter ce type via `create_doc(doc_type="<doc_type>", data=...)`.
>
> Trous identifiés dans le knowledge base (à compléter quand tu veux) :
> - `<liste de ce qui a été noté à l'étape 3>`

## Règles d'or

### 1. Pas de génération silencieuse

Tu ne soumets **aucun bundle** avant que l'utilisateur ait validé son contenu en chat et que `preview_user_template` ait retourné un HTML rendu sans erreur.

### 2. Template abstrait, example concret

Tu ne hardcodes JAMAIS de données prospect (nom client, métriques, URLs) dans le `template.html`. Le template manipule uniquement `data.<champ>`. Toute donnée concrète vit dans l'example (ou dans le draft poussé par `revolucy-doc-draft`).

### 3. Preview avant submit, toujours

`submit_user_template` re-fait la même validation que `preview_user_template`. Toujours preview d'abord pour avoir le feedback structuré.

### 4. Respect strict du knowledge base existant

Le schéma et l'example DOIVENT respecter les règles chargées via `revolucy://context/ton-redactionnel` et `revolucy://context/do-dont`. Avant de proposer un schéma/example, relis ces resources et confronte ton draft à chaque règle.

### 5. Pas de réinvention de la charte

La structure CSS / fonts / palette / blocs réutilisables est figée et décrite dans `revolucy://context/charte-graphique`. Tu réutilises ce qui existe plutôt que de créer du neuf.

### 6. Évite les types qui chevauchent

Avant de créer un type, vérifie qu'il n'est pas couvert (même partiellement) par un type existant via `list_doc_types` puis `validate_doc_type` sur les voisins. Mieux vaut un type bien cadré que deux qui se marchent dessus.

### 7. Form custom : exception, pas règle

Par défaut, **pas de `form.html`**. Le form générique du studio lit `/api/doc_types/<doc_type>` et rend les champs depuis le schéma. Tu ne proposes un form custom que si l'utilisateur a explicitement un besoin que le générique ne couvre pas.

## Gotchas print/CSS (faux overflow et drift preview↔PDF)

Le studio rend deux versions du template : un HTML preview dans l'iframe du form (`@media screen`) et un PDF généré par Chromium headless (`@media print`). Si les deux divergent en spacing, l'indicateur d'overflow rayé rose en preview mentit : il signale un dépassement A4 alors que le PDF tient propre, ou inversement. Ces règles évitent ça.

### 1. Padding `.page` identique en base et en `@media print`

Mets le padding `.page` UNE seule fois, au top-level (hors `@media`). Ne le redéfinis PAS dans `@media print`. Si tu mets `padding: 22mm 20mm 16mm 20mm` en base et `padding: 15mm 18mm 10mm 18mm` en print, le screen render aura 13mm de moins de hauteur utile → faux overflow indicator alors que le PDF fit.

```css
/* ✅ correct */
.page {
  width: 210mm; min-height: 297mm;
  padding: 15mm 18mm 10mm 18mm;
}

/* ❌ ne fais pas ça */
.page { padding: 22mm 20mm 16mm 20mm; }
@media print { .page { padding: 15mm 18mm 10mm 18mm; } }
```

### 2. Spacing (margin/padding/gap/font-size) au top-level, pas dupliqué

Toute valeur qui doit être identique en screen et en print → au top-level. `@media print {}` ne contient QUE des règles print-spécifiques (`page-break-*`, `overflow: hidden`, `box-shadow: none`, `print-color-adjust`). Pas d'override de spacing ou de `font-size`.

```css
/* ✅ correct — spacing partagé */
.constat { padding: 8pt 0; }
.axes { gap: 6pt; }
h1 { font-size: 34pt; }

/* ❌ ne fais pas ça — drift garanti entre preview et PDF */
.constat { padding: 12pt 0; }
@media print { .constat { padding: 8pt 0; } }
```

### 3. Overflow indicator = `.page::after { top: 297mm; bottom: 0 }`

L'indicateur "ça déborde" (zone rayée rose) doit être un pseudo-élément `.page::after` positionné `top: 297mm; bottom: 0`, dans `@media screen` uniquement. Quand la page fait exactement 297mm → 0 de hauteur (invisible). Quand le contenu dépasse → la page grandit et la zone rayée s'étend pour matérialiser l'overflow.

```css
@media screen {
  .page { min-height: 297mm; overflow: visible; }
  .page::after {
    content: ""; position: absolute;
    top: 297mm; left: 0; right: 0; bottom: 0;
    background: repeating-linear-gradient(135deg,
      rgba(251, 0, 101, 0.06) 0, rgba(251, 0, 101, 0.06) 10px,
      transparent 10px, transparent 22px);
    border-top: 1.5pt dashed rgba(251, 0, 101, 0.5);
    pointer-events: none; z-index: 0;
  }
}
```

Ne force JAMAIS `height: 297mm` ou `overflow: hidden` en screen — le clipping empêche la détection. C'est en print uniquement.

### 4. Unités : `pt` pour typo, `mm` pour layout A4

- `font-size`, `padding` interne aux blocs, `gap`, `border-width` → **`pt`**
- `width: 210mm`, `min-height: 297mm`, padding `.page`, margins inter-page → **`mm`**
- Évite `px` et `rem` dans les templates print : ils dépendent du DPI screen et créent du drift entre preview et export.

### 5. `page-break-inside: avoid` sur les blocs non-cassables

Pour les blocs qui ne doivent pas être coupés en deux entre 2 pages (un `.constat`, un `.axe`, un `.closing`, le `.footer`) :

```css
@media print {
  .constat, .axe, .closing, .footer, .header, .hero {
    page-break-inside: avoid;
    break-inside: avoid;
  }
  h1, h2, h3 { page-break-after: avoid; break-after: avoid; }
}
```

Sans ça, un bloc peut être splitté au milieu en PDF → moche.

### 6. Vérification au moment du preview

Avant `submit_user_template`, fais le test suivant :

1. Ouvre `http://localhost:8765/docs/<doc_id>` dans un browser et regarde la preview iframe.
2. Clique **Exporter PDF**, ouvre le PDF.
3. Compare les deux côte à côte sur la même page : ils doivent être identiques au mm près (overflow indicator inclus — s'il s'allume en preview, le contenu doit vraiment être tronqué en PDF).

Si drift : le suspect numéro 1 est un padding `.page` ou un spacing oublié dans `@media print`. Cherche un override print qui n'a pas son équivalent au top-level.

## Effort estimé

~30-45 min pour les 3 artifacts minimum (schema, template, example) si le form générique suffit. +1-3h si form custom nécessaire.
