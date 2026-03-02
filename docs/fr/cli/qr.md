---
summary: "Référence CLI pour `openclaw qr` (générer un QR d'appairage iOS + code de configuration)"
read_when:
  - Vous souhaitez appairer rapidement l'application iOS avec un Gateway
  - Vous avez besoin de la sortie du code de configuration pour un partage distant/manuel
title: "qr"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/qr.md
  workflow: manual
---

# `openclaw qr`

Générer un QR d'appairage iOS et un code de configuration à partir de votre configuration Gateway actuelle.

## Utilisation

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## Options

- `--remote` : utiliser `gateway.remote.url` plus le jeton/mot de passe distant depuis la configuration
- `--url <url>` : surcharger l'URL du Gateway utilisée dans la charge utile
- `--public-url <url>` : surcharger l'URL publique utilisée dans la charge utile
- `--token <token>` : surcharger le jeton du Gateway pour la charge utile
- `--password <password>` : surcharger le mot de passe du Gateway pour la charge utile
- `--setup-code-only` : afficher uniquement le code de configuration
- `--no-ascii` : ignorer le rendu ASCII du QR
- `--json` : émettre du JSON (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## Notes

- `--token` et `--password` sont mutuellement exclusifs.
- Après le scan, approuvez l'appairage de l'appareil avec :
  - `openclaw devices list`
  - `openclaw devices approve <requestId>`
