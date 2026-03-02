---
summary: "Référence CLI pour `openclaw directory` (self, pairs, groupes)"
read_when:
  - Vous souhaitez rechercher des contacts/groupes/identifiants self pour un canal
  - Vous développéz un adaptateur de répertoire de canal
title: "directory"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/directory.md
  workflow: manual
---

# `openclaw directory`

Recherches de répertoire pour les canaux qui le supportent (contacts/pairs, groupes et "me").

## Options communes

- `--channel <name>` : identifiant/alias de canal (requis lorsque plusieurs canaux sont configurés ; automatique lorsqu'un seul est configuré)
- `--account <id>` : identifiant de compte (défaut : défaut du canal)
- `--json` : sortie JSON

## Notes

- `directory` est concu pour vous aider a trouver des identifiants que vous pouvez coller dans d'autres commandes (notamment `openclaw message send --target ...`).
- Pour de nombreux canaux, les résultats sont basés sur la configuration (listes d'autorisation / groupes configurés) plutôt que sur un répertoire fournisseur en direct.
- La sortie par défaut est `id` (et parfois `name`) séparés par une tabulation ; utilisez `--json` pour le scripting.

## Utilisation des résultats avec `message send`

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## Formats d'identifiant (par canal)

- WhatsApp : `+15551234567` (DM), `1234567890-1234567890@g.us` (groupe)
- Telegram : `@username` ou identifiant de chat numérique ; les groupes sont des identifiants numériques
- Slack : `user:U…` et `channel:C…`
- Discord : `user:<id>` et `channel:<id>`
- Matrix (plugin) : `user:@user:server`, `room:!roomId:server` ou `#alias:server`
- Microsoft Teams (plugin) : `user:<id>` et `conversation:<id>`
- Zalo (plugin) : identifiant utilisateur (Bot API)
- Zalo Personal / `zalouser` (plugin) : identifiant de fil (DM/groupe) depuis `zca` (`me`, `friend list`, `group list`)

## Self ("me")

```bash
openclaw directory self --channel zalouser
```

## Pairs (contacts/utilisateurs)

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## Groupes

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```
