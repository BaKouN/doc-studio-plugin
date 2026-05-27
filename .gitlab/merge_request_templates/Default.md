<!-- Template MR par défaut pour le plugin Revolucy Doc Studio.
     Garde-le court : ce repo c'est juste un manifeste marketplace + 2 skills,
     les MR sont rares. -->

## Pourquoi

<!-- 1-2 phrases sur la raison de la modification. -->

## Quoi

<!-- Liste des fichiers touchés et de l'impact attendu côté end-user. -->

## Checklist

- [ ] `CHANGELOG.md` mis à jour sous `[Unreleased]` ou nouvelle version datée
- [ ] `plugin/.claude-plugin/plugin.json` version bumpée si breaking ou nouvelle feature
- [ ] `.claude-plugin/marketplace.json` version alignée si bump plugin
- [ ] Audit IP rapide (`grep` prénoms équipe / prospects réels / noms ressources avant push public)
- [ ] `README.md` ou `ONBOARDING.md` mis à jour si la commande d'install change
