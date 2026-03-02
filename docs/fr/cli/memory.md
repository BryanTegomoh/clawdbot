---
summary: "Référence CLI pour `openclaw memory` (statut/indexation/recherche)"
read_when:
  - Vous souhaitez indexer ou rechercher dans la mémoire semantique
  - Vous déboguez la disponibilité de la mémoire ou l'indexation
title: "memory"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/memory.md
  workflow: manual
---

# `openclaw memory`

Gérer l'indexation et la recherche de mémoire semantique.
Fourni par le plugin mémoire actif (défaut : `memory-core` ; définissez `plugins.slots.memory = "none"` pour désactiver).

Liens :

- Concept de mémoire : [Memory](/concepts/memory)
- Plugins : [Plugins](/tools/plugin)

## Exemples

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory search --query "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## Options

Communes :

- `--agent <id>` : limiter à un seul agent (défaut : tous les agents configurés).
- `--verbose` : émettre des logs détaillés pendant les sondes et l'indexation.

`memory search` :

- Entrée de requête : passez soit un `[query]` positionnel soit `--query <text>`.
- Si les deux sont fournis, `--query` l'emporte.
- Si aucun n'est fourni, la commande quitte avec une erreur.

Notes :

- `memory status --deep` sonde la disponibilité vectorielle + embedding.
- `memory status --deep --index` exécute une reindexation si le magasin est modifie.
- `memory index --verbose` affiche les détails par phase (fournisseur, modèle, sources, activité par lots).
- `memory status` inclut tout chemin supplémentaire configuré via `memorySearch.extraPaths`.
