---
summary: "Référence CLI pour `openclaw status` (diagnostics, sondes, instantanés d'utilisation)"
read_when:
  - Vous souhaitez un diagnostic rapide de la santé des canaux + destinataires de sessions récentes
  - Vous souhaitez un statut « all » à copier-coller pour le débogage
title: "status"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/status.md
  workflow: manual
---

# `openclaw status`

Diagnostics pour les canaux + sessions.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notes :

- `--deep` exécute des sondes en direct (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
- La sortie inclut les magasins de sessions par agent quand plusieurs agents sont configurés.
- La vue d'ensemble inclut le statut d'installation/d'exécution du service hôte Gateway + nœud quand disponible.
- La vue d'ensemble inclut le canal de mise à jour + le SHA git (pour les checkouts source).
- Les informations de mise à jour apparaissent dans la vue d'ensemble ; si une mise à jour est disponible, status affiche un indice pour exécuter `openclaw update` (voir [Mise à jour](/install/updating)).
