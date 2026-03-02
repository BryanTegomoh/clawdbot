---
summary: "Référence CLI pour `openclaw pairing` (approuver/lister les demandes d'appairage)"
read_when:
  - Vous utilisez les DM en mode appairage et devez approuver des expéditeurs
title: "pairing"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/pairing.md
  workflow: manual
---

# `openclaw pairing`

Approuver ou inspecter les demandes d'appairage DM (pour les canaux qui supportent l'appairage).

Liens :

- Flux d'appairage : [Pairing](/channels/pairing)

## Commandes

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## Notes

- Entrée de canal : passez-la positionnellement (`pairing list telegram`) ou avec `--channel <channel>`.
- `pairing list` supporte `--account <accountId>` pour les canaux multi-comptes.
- `pairing approve` supporte `--account <accountId>` et `--notify`.
- Si un seul canal compatible avec l'appairage est configuré, `pairing approve <code>` est autorisé.
