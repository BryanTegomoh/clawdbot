---
summary: "Référence CLI pour `openclaw logs` (suivre les logs du Gateway via RPC)"
read_when:
  - Vous devez suivre les logs du Gateway à distance (sans SSH)
  - Vous souhaitez des lignes de log JSON pour l'outillage
title: "logs"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/logs.md
  workflow: manual
---

# `openclaw logs`

Suivre les logs du fichier Gateway via RPC (fonctionne en mode distant).

Liens :

- Aperçu de la journalisation : [Logging](/logging)

## Exemples

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

Utilisez `--local-time` pour afficher les horodatages dans votre fuseau horaire local.
