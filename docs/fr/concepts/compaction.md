---
summary: "Fenêtre de contexte + compaction : comment OpenClaw maintient les sessions sous les limités du modèle"
read_when:
  - Vous souhaitez comprendre la compaction automatique et /compact
  - Vous déboguez des sessions longues qui atteignent les limités de contexte
title: "Compaction"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/compaction.md
  workflow: manual
---

# Fenêtre de contexte et compaction

Chaque modèle possède une **fenêtre de contexte** (nombre maximum de tokens qu'il peut voir). Les conversations longues accumulent des messages et des résultats d'outils ; une fois la fenêtre serrée, OpenClaw **compacte** l'historique ancien pour rester dans les limités.

## Qu'est-ce que la compaction

La compaction **résume la conversation plus ancienne** en une entrée de résumé compact et conserve les messages récents intacts. Le résumé est stocké dans l'historique de session, de sorte que les requêtes futures utilisent :

- Le résumé de compaction
- Les messages récents après le point de compaction

La compaction **persiste** dans l'historique JSONL de la session.

## Configuration

Utilisez le paramètre `agents.defaults.compaction` dans votre `openclaw.json` pour configurer le comportement de compaction (mode, tokens cible, etc.).

## Compaction automatique (activée par défaut)

Lorsqu'une session approche ou dépasse la fenêtre de contexte du modèle, OpenClaw déclenche la compaction automatique et peut réessayer la requête originale en utilisant le contexte compacté.

Vous verrez :

- `🧹 Auto-compaction complète` en mode verbose
- `/status` affichant `🧹 Compactions: <nombre>`

Avant la compaction, OpenClaw peut exécuter un tour de **purge silencieuse de mémoire** pour stocker des notes durables sur le disque. Voir [Mémoire](/concepts/memory) pour les détails et la configuration.

## Compaction manuelle

Utilisez `/compact` (optionnellement avec des instructions) pour forcer une passe de compaction :

```
/compact Focus on decisions and open questions
```

## Source de la fenêtre de contexte

La fenêtre de contexte est spécifique au modèle. OpenClaw utilise la définition du modèle depuis le catalogue du fournisseur configuré pour déterminer les limités.

## Compaction vs élagage

- **Compaction** : résume et **persiste** dans le JSONL.
- **Élagage de session** : supprimé les anciens **résultats d'outils** uniquement, **en mémoire**, par requête.

Voir [/concepts/session-pruning](/concepts/session-pruning) pour les détails de l'élagage.

## Conseils

- Utilisez `/compact` quand les sessions semblent obsolètes ou que le contexte est gonflé.
- Les sorties d'outils volumineuses sont déjà tronquées ; l'élagage peut réduire davantage l'accumulation de résultats d'outils.
- Si vous avez besoin d'un nouveau départ, `/new` ou `/reset` démarre un nouvel identifiant de session.
