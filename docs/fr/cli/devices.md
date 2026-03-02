---
summary: "Référence CLI pour `openclaw devices` (appairage d'appareils + rotation/révocation de jetons)"
read_when:
  - Vous approuvez des demandes d'appairage d'appareils
  - Vous devez effectuer une rotation ou revoquer des jetons d'appareils
title: "devices"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/devices.md
  workflow: manual
---

# `openclaw devices`

Gérer les demandes d'appairage d'appareils et les jetons limités aux appareils.

## Commandes

### `openclaw devices list`

Lister les demandes d'appairage en attente et les appareils appairés.

```
openclaw devices list
openclaw devices list --json
```

### `openclaw devices remove <deviceId>`

Supprimer une entrée d'appareil appaire.

```
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### `openclaw devices clear --yes [--pending]`

Effacer les appareils appairés en masse.

```
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### `openclaw devices approve [requestId] [--latest]`

Approuver une demande d'appairage d'appareil en attente. Si `requestId` est omis, OpenClaw approuve automatiquement la demande en attente la plus récente.

```
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### `openclaw devices reject <requestId>`

Rejeter une demande d'appairage d'appareil en attente.

```
openclaw devices reject <requestId>
```

### `openclaw devices rotate --device <id> --rôle <rôle> [--scope <scope...>]`

Effectuer une rotation du jeton d'appareil pour un rôle spécifique (avec mise à jour optionnelle des portées).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### `openclaw devices revoke --device <id> --rôle <rôle>`

Révoquer un jeton d'appareil pour un rôle spécifique.

```
openclaw devices revoke --device <deviceId> --role node
```

## Options communes

- `--url <url>` : URL WebSocket du Gateway (par défaut `gateway.remote.url` lorsque configuré).
- `--token <token>` : jeton du Gateway (si requis).
- `--password <password>` : mot de passe du Gateway (authentification par mot de passe).
- `--timeout <ms>` : délai d'expiration RPC.
- `--json` : sortie JSON (recommandé pour le scripting).

Note : lorsque vous définissez `--url`, le CLI ne se replie pas sur les identifiants de la configuration ou de l'environnement.
Passez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur.

## Notes

- La rotation de jeton renvoie un nouveau jeton (sensible). Traitez-le comme un secret.
- Ces commandes nécessitent la portée `operator.pairing` (ou `operator.admin`).
- `devices clear` est intentionnellement protégé par `--yes`.
- Si la portée d'appairage n'est pas disponible sur le loopback local (et qu'aucun `--url` explicite n'est passé), list/approve peut utiliser un repli d'appairage local.
