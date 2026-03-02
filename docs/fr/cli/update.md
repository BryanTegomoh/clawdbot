---
summary: "Référence CLI pour `openclaw update` (mise à jour source sécurisée + redémarrage automatique du Gateway)"
read_when:
  - Vous souhaitez mettre à jour un checkout source en toute sécurité
  - Vous avez besoin de comprendre le comportement du raccourci `--update`
title: "update"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/update.md
  workflow: manual
---

# `openclaw update`

Mettre à jour OpenClaw en toute sécurité et basculer entre les canaux stable/beta/dev.

Si vous avez installé via **npm/pnpm** (installation globale, pas de métadonnées git), les mises à jour se font via le flux du gestionnaire de paquets dans [Mise à jour](/install/updating).

## Utilisation

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --dry-run
openclaw update --no-restart
openclaw update --json
openclaw --update
```

## Options

- `--no-restart` : ne pas redémarrer le service Gateway après une mise à jour réussie.
- `--channel <stable|beta|dev>` : définir le canal de mise à jour (git + npm ; persisté dans la configuration).
- `--tag <dist-tag|version>` : remplacer le dist-tag ou la version npm pour cette mise à jour uniquement.
- `--dry-run` : prévisualiser les actions de mise à jour planifiées (canal/tag/cible/flux de redémarrage) sans écrire la configuration, installer, synchroniser les plugins, ni redémarrer.
- `--json` : afficher le JSON `UpdateRunResult` lisible par machine.
- `--timeout <seconds>` : délai d'attente par étape (par défaut 1200 s).

Note : les rétrogradations nécessitent une confirmation car les versions antérieures peuvent casser la configuration.

## `update status`

Afficher le canal de mise à jour actif + le tag/branche/SHA git (pour les checkouts source), plus la disponibilité des mises à jour.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Options :

- `--json` : afficher le statut JSON lisible par machine.
- `--timeout <seconds>` : délai d'attente pour les vérifications (par défaut 3 s).

## `update wizard`

Flux interactif pour choisir un canal de mise à jour et confirmer s'il faut redémarrer le Gateway
après la mise à jour (le redémarrage est le comportement par défaut). Si vous sélectionnez `dev` sans checkout git, il
propose d'en créer un.

## Ce qu'il fait

Quand vous changez de canal explicitement (`--channel ...`), OpenClaw maintient également la
méthode d'installation alignée :

- `dev` -> s'assuré qu'un checkout git existe (par défaut : `~/openclaw`, remplacez avec `OPENCLAW_GIT_DIR`),
  le met à jour et installe le CLI global depuis ce checkout.
- `stable`/`beta` -> installe depuis npm en utilisant le dist-tag correspondant.

La mise à jour automatique du cœur du Gateway (quand activée via la configuration) réutilise ce même chemin de mise à jour.

## Flux de checkout git

Canaux :

- `stable` : checkout du dernier tag non-beta, puis build + doctor.
- `beta` : checkout du dernier tag `-beta`, puis build + doctor.
- `dev` : checkout de `main`, puis fetch + rebase.

Vue d'ensemble :

1. Nécessite un répertoire de travail propre (pas de modifications non commitées).
2. Bascule vers le canal sélectionné (tag ou branche).
3. Fetch de l'upstream (dev uniquement).
4. Dev uniquement : vérification préliminaire lint + build TypeScript dans un répertoire de travail temporaire ; si la pointe échoue, remonte jusqu'à 10 commits pour trouver le build propre le plus récent.
5. Rebase sur le commit sélectionné (dev uniquement).
6. Installation des dépendances (pnpm préféré ; npm en repli).
7. Build + construction de l'interface de contrôle.
8. Exécution de `openclaw doctor` comme vérification finale de « mise à jour sûre ».
9. Synchronisation des plugins vers le canal actif (dev utilisé les extensions intégrées ; stable/beta utilisé npm) et mise à jour des plugins installés via npm.

## Raccourci `--update`

`openclaw --update` est réécrit en `openclaw update` (utile pour les shells et scripts de lancement).

## Voir aussi

- `openclaw doctor` (propose d'exécuter update d'abord sur les checkouts git)
- [Canaux de développement](/install/development-channels)
- [Mise à jour](/install/updating)
- [Référence CLI](/cli)
