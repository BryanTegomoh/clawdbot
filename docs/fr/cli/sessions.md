---
summary: "Référence CLI pour `openclaw sessions` (lister les sessions stockées + utilisation)"
read_when:
  - Vous souhaitez lister les sessions stockées et voir l'activité récente
title: "sessions"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/sessions.md
  workflow: manual
---

# `openclaw sessions`

Lister les sessions de conversation stockées.

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

Sélection de la portée :

- par défaut : magasin de l'agent par défaut configuré
- `--agent <id>` : un magasin d'agent configuré
- `--all-agents` : agrège tous les magasins d'agents configurés
- `--store <path>` : chemin de magasin explicite (ne peut pas être combiné avec `--agent` ou `--all-agents`)

Exemples JSON :

`openclaw sessions --all-agents --json` :

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## Maintenance de nettoyage

Exécuter la maintenance maintenant (au lieu d'attendre le prochain cycle d'écriture) :

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

`openclaw sessions cleanup` utilisé les paramètres `session.maintenance` de la configuration :

- Note sur la portée : `openclaw sessions cleanup` maintient uniquement les magasins/transcriptions de sessions. Il n'élague pas les journaux d'exécution cron (`cron/runs/<jobId>.jsonl`), qui sont gérés par `cron.runLog.maxBytes` et `cron.runLog.keepLines` dans la [configuration Cron](/automation/cron-jobs#configuration) et expliqués dans la [maintenance Cron](/automation/cron-jobs#maintenance).

- `--dry-run` : prévisualiser combien d'entrées seraient élaguées/plafonnées sans écriture.
  - En mode texte, dry-run affiche un tableau d'actions par session (`Action`, `Key`, `Age`, `Model`, `Flags`) pour que vous puissiez voir ce qui serait conservé vs supprimé.
- `--enforce` : appliquer la maintenance même quand `session.maintenance.mode` est `warn`.
- `--activated-key <key>` : protéger une clé activé spécifique de l'éviction par budget disque.
- `--agent <id>` : exécuter le nettoyage pour un magasin d'agent configuré.
- `--all-agents` : exécuter le nettoyage pour tous les magasins d'agents configurés.
- `--store <path>` : exécuter sur un fichier `sessions.json` spécifique.
- `--json` : afficher un résumé JSON. Avec `--all-agents`, la sortie inclut un résumé par magasin.

`openclaw sessions cleanup --all-agents --dry-run --json` :

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

Liens connexes :

- Configuration des sessions : [Référence de configuration](/gateway/configuration-reference#session)
