---
summary: "Validation stricte de la config + migrations par doctor uniquement"
read_when:
  - Conception ou implémentation du comportement de validation de config
  - Travail sur les migrations de config ou les workflows doctor
  - Gestion des schémas de config plugin ou du gating de chargement de plugin
title: "Validation stricte de la configuration"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/refactor/strict-config.md
  workflow: manual
---

# Validation stricte de la configuration (migrations par doctor uniquement)

## Objectifs

- **Rejeter les clés de config inconnues partout** (racine + imbriqué), sauf les métadonnées racine `$schema`.
- **Rejeter la config plugin sans schéma** ; ne pas charger ce plugin.
- **Supprimer la migration automatique legacy au chargement** ; les migrations s'exécutent via doctor uniquement.
- **Exécuter automatiquement doctor (dry-run) au démarrage** ; si invalide, bloquer les commandes non-diagnostiques.

## Hors périmètre

- Rétrocompatibilité au chargement (les clés legacy ne migrent pas automatiquement).
- Suppressions silencieuses de clés non reconnues.

## Règles de validation stricte

- La config doit correspondre exactement au schéma à chaque niveau.
- Les clés inconnues sont des erreurs de validation (pas de passthrough à la racine ou en imbriqué), sauf `$schema` racine quand c'est une chaîne.
- `plugins.entries.<id>.config` doit être validé par le schéma du plugin.
  - Si un plugin n'a pas de schéma, **rejeter le chargement du plugin** et afficher une erreur claire.
- Les clés `channels.<id>` inconnues sont des erreurs sauf si un manifeste de plugin déclare l'id de canal.
- Les manifestes de plugin (`openclaw.plugin.json`) sont requis pour tous les plugins.

## Application du schéma plugin

- Chaque plugin fournit un JSON Schema strict pour sa config (inline dans le manifeste).
- Flux de chargement de plugin :
  1. Résoudre le manifeste + schéma du plugin (`openclaw.plugin.json`).
  2. Valider la config contre le schéma.
  3. Si schéma manquant ou config invalide : bloquer le chargement du plugin, enregistrer l'erreur.
- Le message d'erreur inclut :
  - L'id du plugin
  - La raison (schéma manquant / config invalide)
  - Le(s) chemin(s) qui ont échoué à la validation
- Les plugins désactivés conservent leur config, mais Doctor + les logs affichent un avertissement.

## Flux Doctor

- Doctor s'exécute **à chaque fois** que la config est chargée (dry-run par défaut).
- Si la config est invalide :
  - Afficher un résumé + erreurs actionnables.
  - Instruire : `openclaw doctor --fix`.
- `openclaw doctor --fix` :
  - Applique les migrations.
  - Supprimé les clés inconnues.
  - Écrit la config mise à jour.

## Gating de commande (quand la config est invalide)

Autorisé (diagnostic uniquement) :

- `openclaw doctor`
- `openclaw logs`
- `openclaw health`
- `openclaw help`
- `openclaw status`
- `openclaw gateway status`

Tout le reste doit échouer immédiatement avec : « Config invalide. Exécutez `openclaw doctor --fix`. »

## Format UX des erreurs

- En-tête de résumé unique.
- Sections groupées :
  - Clés inconnues (chemins complets)
  - Clés legacy / migrations nécessaires
  - Échecs de chargement de plugin (id du plugin + raison + chemin)

## Points de contact d'implémentation

- `src/config/zod-schema.ts` : supprimer le passthrough racine ; objets stricts partout.
- `src/config/zod-schema.providers.ts` : assurer des schémas de canal stricts.
- `src/config/validation.ts` : échouer sur les clés inconnues ; ne pas appliquer les migrations legacy.
- `src/config/io.ts` : supprimer les migrations automatiques legacy ; toujours exécuter le dry-run doctor.
- `src/config/legacy*.ts` : déplacer l'utilisation vers doctor uniquement.
- `src/plugins/*` : ajouter le registre de schémas + gating.
- Gating de commande CLI dans `src/cli`.

## Tests

- Rejet de clé inconnue (racine + imbriqué).
- Plugin sans schéma -> chargement du plugin bloqué avec erreur claire.
- Config invalide -> démarrage du Gateway bloqué sauf commandes de diagnostic.
- Dry-run automatique de Doctor ; `doctor --fix` écrit la config corrigée.
