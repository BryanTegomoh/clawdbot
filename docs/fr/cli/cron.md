---
summary: "Référence CLI pour `openclaw cron` (planifier et exécuter des tâches en arrière-plan)"
read_when:
  - Vous souhaitez des tâches planifiées et des réveils
  - Vous déboguez l'exécution et les logs cron
title: "cron"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/cron.md
  workflow: manual
---

# `openclaw cron`

Gérer les tâches cron pour le planificateur du Gateway.

Liens :

- Tâches cron : [Cron jobs](/automation/cron-jobs)

Astuce : exécutez `openclaw cron --help` pour la surface de commande complète.

Note : les tâches isolées `cron add` utilisent par défaut la livraison `--announce`. Utilisez `--no-deliver` pour garder la sortie interne. `--deliver` reste un alias obsolète pour `--announce`.

Note : les tâches ponctuelles (`--at`) se suppriment après le succès par défaut. Utilisez `--keep-after-run` pour les conserver.

Note : les tâches récurrentes utilisent désormais un backoff exponentiel de relance après des erreurs consécutives (30s, 1m, 5m, 15m, 60m), puis reviennent au calendrier normal après la prochaine exécution réussie.

Note : la rétention/l'élagage est contrôlé dans la configuration :

- `cron.sessionRetention` (défaut `24h`) élague les sessions d'exécution isolées terminées.
- `cron.runLog.maxBytes` + `cron.runLog.keepLines` élaguent `~/.openclaw/cron/runs/<jobId>.jsonl`.

## Modifications courantes

Mettre à jour les paramètres de livraison sans changer le message :

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

Désactiver la livraison pour une tâche isolée :

```bash
openclaw cron edit <job-id> --no-deliver
```

Annoncer sur un canal spécifique :

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```
