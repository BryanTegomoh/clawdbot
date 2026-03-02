---
summary: "Référence CLI pour `openclaw nodes` (lister/statut/approuver/invoquer, camera/canvas/écran)"
read_when:
  - Vous gérez des nodes appairés (cameras, écran, canvas)
  - Vous devez approuver des demandes ou invoquer des commandes de node
title: "nodes"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/nodes.md
  workflow: manual
---

# `openclaw nodes`

Gérer les nodes appairés (appareils) et invoquer les capacités des nodes.

Liens :

- Aperçu des nodes : [Nodes](/nodes)
- Camera : [Camera nodes](/nodes/camera)
- Images : [Image nodes](/nodes/images)

Options communes :

- `--url`, `--token`, `--timeout`, `--json`

## Commandes courantes

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` affiche les tableaux en attente/appairés. Les lignes appairées incluent l'ancienneté de la dernière connexion (Last Connect).
Utilisez `--connected` pour afficher uniquement les nodes actuellement connectés. Utilisez `--last-connected <duration>` pour filtrer les nodes qui se sont connectés dans un délai (par ex. `24h`, `7d`).

## Invoquer / exécuter

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Options d'invocation :

- `--params <json>` : chaine d'objet JSON (défaut `{}`).
- `--invoke-timeout <ms>` : délai d'expiration d'invocation du node (défaut `15000`).
- `--idempotency-key <key>` : clé d'idempotence optionnelle.

### Comportements par défaut de type exec

`nodes run` reproduit le comportement exec du modèle (valeurs par défaut + approbations) :

- Lit `tools.exec.*` (plus les surcharges `agents.list[].tools.exec.*`).
- Utilisé les approbations d'exécution (`exec.approval.request`) avant d'invoquer `system.run`.
- `--node` peut être omis lorsque `tools.exec.node` est défini.
- Nécessite un node qui annonce `system.run` (application macOS ou node host headless).

Options :

- `--cwd <path>` : répertoire de travail.
- `--env <key=val>` : surcharge d'environnement (repetable). Note : les node hosts ignorent les surcharges de `PATH` (et `tools.exec.pathPrepend` n'est pas applique aux node hosts).
- `--command-timeout <ms>` : délai d'expiration de la commande.
- `--invoke-timeout <ms>` : délai d'expiration d'invocation du node (défaut `30000`).
- `--needs-screen-recording` : nécessiter l'autorisation d'enregistrement d'écran.
- `--raw <command>` : exécuter une chaine shell (`/bin/sh -lc` ou `cmd.exe /c`).
  En mode liste d'autorisation sur les node hosts Windows, les exécutions enveloppées par `cmd.exe /c` nécessitent une approbation (l'entrée de la liste d'autorisation seule n'autorisé pas automatiquement la forme enveloppée).
- `--agent <id>` : approbations/listes d'autorisation limitées à l'agent (par défaut l'agent configure).
- `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>` : surcharges.
