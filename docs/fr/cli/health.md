---
summary: "Référence CLI pour `openclaw health` (endpoint de santé du Gateway via RPC)"
read_when:
  - Vous souhaitez vérifier rapidement la santé du Gateway en cours d'exécution
title: "health"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/health.md
  workflow: manual
---

# `openclaw health`

Récupérer la santé du Gateway en cours d'exécution.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notes :

- `--verbose` exécute des sondes en direct et affiche les temps par compte lorsque plusieurs comptes sont configurés.
- La sortie inclut les magasins de sessions par agent lorsque plusieurs agents sont configurés.
