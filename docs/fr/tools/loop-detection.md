---
title: "Détection de boucles d'outils"
description: "Configurer des garde-fous optionnels pour prévenir les boucles d'appels d'outils répétitifs ou bloqués"
summary: "Comment activer et ajuster les garde-fous qui détectent les boucles d'appels d'outils répétitifs"
read_when:
  - Un utilisateur signale que les agents restent bloqués en répétant des appels d'outils
  - Vous devez ajuster la protection contre les appels répétitifs
  - Vous modifiez les politiques d'outils/runtime des agents
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/loop-détection.md
  workflow: manual
---

# Détection de boucles d'outils

OpenClaw peut empêcher les agents de rester bloqués dans des schémas d'appels d'outils répétés.
Le garde-fou est **désactivé par défaut**.

Activez-le uniquement là où c'est nécessaire, car il peut bloquer des appels répétés légitimes avec des paramètres stricts.

## Pourquoi cela existe

- Détecter les séquences répétitives qui ne progressent pas.
- Détecter les boucles sans résultat à haute fréquence (même outil, mêmes entrées, erreurs répétées).
- Détecter des schémas d'appels répétés spécifiques pour des outils de polling connus.

## Bloc de configuration

Valeurs par défaut globales :

```json5
{
  tools: {
    loopDetection: {
      enabled: false,
      historySize: 20,
      detectorCooldownMs: 12000,
      repeatThreshold: 3,
      criticalThreshold: 6,
      detectors: {
        repeatedFailure: true,
        knownPollLoop: true,
        repeatingNoProgress: true,
      },
    },
  },
}
```

Remplacement par agent (optionnel) :

```json5
{
  agents: {
    list: [
      {
        id: "safe-runner",
        tools: {
          loopDetection: {
            enabled: true,
            repeatThreshold: 2,
            criticalThreshold: 5,
          },
        },
      },
    ],
  },
}
```

### Comportement des champs

- `enabled` : interrupteur principal. `false` signifie qu'aucune détection de boucle n'est effectuée.
- `historySize` : nombre d'appels d'outils récents conservés pour l'analyse.
- `detectorCooldownMs` : fenêtre temporelle utilisée par le détecteur de non-progression.
- `repeatThreshold` : répétitions minimales avant le début des avertissements/blocages.
- `criticalThreshold` : seuil plus strict qui peut déclencher un traitement plus sévère.
- `detectors.repeatedFailure` : détecte les tentatives échouées répétées sur le même chemin d'appel.
- `detectors.knownPollLoop` : détecte les boucles de type polling connues.
- `detectors.repeatingNoProgress` : détecte les appels répétés à haute fréquence sans changement d'état.

## Configuration recommandée

- Commencez avec `enabled: true`, valeurs par défaut inchangées.
- Si des faux positifs surviennent :
  - augmentez `repeatThreshold` et/ou `criticalThreshold`
  - désactivez uniquement le détecteur qui pose problème
  - réduisez `historySize` pour un contexte historique moins strict

## Journaux et comportement attendu

Lorsqu'une boucle est détectée, OpenClaw signale un événement de boucle et bloque ou atténue le prochain cycle d'outils selon la gravité.
Cela protège les utilisateurs contre les dépenses de tokens incontrôlées et les blocages tout en préservant l'accès normal aux outils.

- Préférez d'abord l'avertissement et la suppression temporaire.
- N'escaladez que lorsque des preuves répétées s'accumulent.

## Notes

- `tools.loopDetection` est fusionné avec les remplacements au niveau de l'agent.
- La configuration par agent remplace ou étend entièrement les valeurs globales.
- Si aucune configuration n'existe, les garde-fous restent désactivés.
