---
summary: "Logique de statut de la barre de menus et informations presentees aux utilisateurs"
read_when:
  - Modification de l'UI du menu mac ou de la logique de statut
title: "Barre de menus"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/menu-bar.md
  workflow: manual
---

# Logique de statut de la barre de menus

## Ce qui est affiche

- Nous affichons l'état de travail actuel de l'agent dans l'icone de la barre de menus et dans la première ligne de statut du menu.
- Le statut de sante est masque pendant que le travail est actif ; il revient lorsque toutes les sessions sont inactives.
- Le bloc « Nodes » dans le menu liste les **appareils** uniquement (nœuds appairés via `node.list`), pas les entrées client/presence.
- Une section « Usage » apparaît sous Context lorsque des instantanes d'utilisation du fournisseur sont disponibles.

## Modèle d'état

- Sessions : les événements arrivent avec `runId` (par exécution) plus `sessionKey` dans la charge utile. La session « main » est la clé `main` ; si absente, on se rabat sur la session mise a jour le plus recemment.
- Priorité : main gagne toujours. Si main est activé, son état est affiche immédiatement. Si main est inactive, la session non-main la plus recemment activé est affichee. Nous ne basculons pas en pleine activité ; nous ne changeons que lorsque la session actuelle devient inactive ou que main devient activé.
- Types d'activité :
  - `job` : exécution de commande de haut niveau (`state: started|streaming|done|error`).
  - `tool` : `phase: start|result` avec `toolName` et `meta/args`.

## Enum IconState (Swift)

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (surcharge de debogage)

### ActivityKind → glyphe

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- défaut → 🛠️

### Correspondance visuelle

- `idle` : mascotte normale.
- `workingMain` : badge avec glyphe, teinte complète, animation « travail » des pattes.
- `workingOther` : badge avec glyphe, teinte attenuee, pas d'agitation.
- `overridden` : utilisé le glyphe/teinte choisi independamment de l'activité.

## Texte de la ligne de statut (menu)

- Pendant le travail actif : `<Rôle de session> · <label d'activité>`
  - Exemples : `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Au repos : se rabat sur le résumé de sante.

## Ingestion des événements

- Source : événements `agent` du canal de contrôle (`ControlChannel.handleAgentEvent`).
- Champs analyses :
  - `stream: "job"` avec `data.state` pour le démarrage/arrêt.
  - `stream: "tool"` avec `data.phase`, `name`, `meta`/`args` optionnels.
- Labels :
  - `exec` : première ligne de `args.command`.
  - `read`/`write` : chemin raccourci.
  - `edit` : chemin plus type de changement infere depuis `meta`/compteurs de diff.
  - repli : nom de l'outil.

## Surcharge de debogage

- Paramètres ▸ Debug ▸ sélecteur « Icon override » :
  - `System (auto)` (par défaut)
  - `Working: main` (par type d'outil)
  - `Working: other` (par type d'outil)
  - `Idle`
- Stocke via `@AppStorage("iconOverride")` ; mappe sur `IconState.overridden`.

## Checklist de test

- Déclencher une tache de la session main : vérifier que l'icone bascule immédiatement et que la ligne de statut affiche le label main.
- Déclencher une tache de session non-main pendant que main est inactive : l'icone/statut affiche non-main ; reste stable jusqu'a la fin.
- Démarrer main pendant qu'une autre est activé : l'icone bascule instantanement vers main.
- Rafales d'outils rapides : s'assurer que le badge ne scintille pas (TTL de grace sur les résultats d'outils).
- La ligne de sante reapparait une fois toutes les sessions inactives.
