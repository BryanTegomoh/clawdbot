---
summary: "Appliquer des patches multi-fichiers avec l'outil apply_patch"
read_when:
  - Vous avez besoin d'éditions structurées sur plusieurs fichiers
  - Vous souhaitez documenter ou déboguer des éditions basées sur des patches
title: "Outil apply_patch"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/apply-patch.md
  workflow: manual
---

# Outil apply_patch

Appliquer des modifications de fichiers en utilisant un format de patch structuré. Idéal pour les éditions
multi-fichiers ou multi-hunk où un seul appel `edit` serait fragile.

L'outil accepte une seule chaîne `input` qui encapsule une ou plusieurs opérations de fichier :

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## Paramètres

- `input` (obligatoire) : Contenu complet du patch incluant `*** Begin Patch` et `*** End Patch`.

## Notes

- Les chemins du patch prennent en charge les chemins relatifs (depuis le répertoire du workspace) et les chemins absolus.
- `tools.exec.applyPatch.workspaceOnly` est défini à `true` par défaut (contenu dans le workspace). Définissez-le à `false` uniquement si vous souhaitez intentionnellement que `apply_patch` écrive/supprimé en dehors du répertoire du workspace.
- Utilisez `*** Move to:` dans un hunk `*** Update File:` pour renommer des fichiers.
- `*** End of File` marque une insertion en fin de fichier uniquement quand nécessaire.
- Expérimental et désactive par défaut. Activer avec `tools.exec.applyPatch.enabled`.
- OpenAI uniquement (y compris OpenAI Codex). Optionnellement limiter par modèle via
  `tools.exec.applyPatch.allowModels`.
- La configuration se trouve uniquement sous `tools.exec`.

## Exemple

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```
