---
title: IRC
description: Connecter OpenClaw aux canaux IRC et aux messages privés.
summary: "Configuration du plugin IRC, contrôles d'accès et dépannage"
read_when:
  - Vous souhaitez connecter OpenClaw à des canaux IRC ou des Messages privés
  - Vous configurez les listes autorisées IRC, la politique de groupes ou le filtrage par mention
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/irc.md
  workflow: manual
---

Utilisez IRC lorsque vous souhaitez qu'OpenClaw soit présent dans les canaux classiques (`#room`) et les messages privés.
IRC est livré comme un plugin d'extension, mais il est configuré dans la configuration principale sous `channels.irc`.

## Démarrage rapide

1. Activez la configuration IRC dans `~/.openclaw/openclaw.json`.
2. Définissez au minimum :

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Démarrez/redémarrez le Gateway :

```bash
openclaw gateway run
```

## Paramètres de sécurité par défaut

- `channels.irc.dmPolicy` est défini par défaut sur `"pairing"`.
- `channels.irc.groupPolicy` est défini par défaut sur `"allowlist"`.
- Avec `groupPolicy="allowlist"`, définissez `channels.irc.groups` pour spécifier les canaux autorisés.
- Utilisez TLS (`channels.irc.tls=true`) sauf si vous acceptez intentionnellement un transport en texte clair.

## Contrôle d'accès

Il existe deux « portes » distinctes pour les canaux IRC :

1. **Accès au canal** (`groupPolicy` + `groups`) : si le bot accepte les messages d'un canal donné.
2. **Accès de l'expéditeur** (`groupAllowFrom` / par canal `groups["#channel"].allowFrom`) : qui est autorisé à déclencher le bot dans ce canal.

Clés de configuration :

- Liste autorisée des Messages privés (accès expéditeur MP) : `channels.irc.allowFrom`
- Liste autorisée des expéditeurs de groupes (accès expéditeur canal) : `channels.irc.groupAllowFrom`
- Contrôles par canal (canal + expéditeur + règles de mention) : `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` autorisé les canaux non configurés (**toujours filtré par mention par défaut**)

Les entrées de la liste autorisée doivent utiliser des identités d'expéditeur stables (`nick!user@host`).
La correspondance par pseudonyme seul est mutable et n'est activée que lorsque `channels.irc.dangerouslyAllowNameMatching: true`.

### Piège courant : `allowFrom` est pour les MP, pas les canaux

Si vous voyez des logs comme :

- `irc: drop group sender alice!ident@host (policy=allowlist)`

...cela signifie que l'expéditeur n'était pas autorisé pour les messages de **groupe/canal**. Corrigez en :

- définissant `channels.irc.groupAllowFrom` (global pour tous les canaux), ou
- définissant des listes autorisées d'expéditeurs par canal : `channels.irc.groups["#channel"].allowFrom`

Exemple (autoriser quiconque dans `#tuirc-dev` à parler au bot) :

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Déclenchement de réponse (mentions)

Même si un canal est autorisé (via `groupPolicy` + `groups`) et que l'expéditeur est autorisé, OpenClaw applique par défaut le **filtrage par mention** dans les contextes de groupe.

Cela signifie que vous pouvez voir des logs comme `drop channel … (missing-mention)` sauf si le message inclut un motif de mention correspondant au bot.

Pour que le bot réponde dans un canal IRC **sans nécessiter de mention**, désactivez le filtrage par mention pour ce canal :

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Ou pour autoriser **tous** les canaux IRC (sans liste autorisée par canal) et répondre sans mentions :

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Note de sécurité (recommandé pour les canaux publics)

Si vous autorisez `allowFrom: ["*"]` dans un canal public, n'importe qui peut solliciter le bot.
Pour réduire les risques, restreignez les outils pour ce canal.

### Mêmes outils pour tous dans le canal

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Outils différents par expéditeur (le propriétaire a plus de droits)

Utilisez `toolsBySender` pour appliquer une politique plus stricte à `"*"` et une plus permissive à votre pseudonyme :

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            "id:eigen": {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Notes :

- Les clés `toolsBySender` doivent utiliser `id:` pour les valeurs d'identité d'expéditeur IRC :
  `id:eigen` ou `id:eigen!~eigen@174.127.248.171` pour une correspondance plus forte.
- Les clés héritées sans préfixe sont toujours acceptées et correspondent uniquement à `id:`.
- La première politique d'expéditeur correspondante gagne ; `"*"` est le repli générique.

Pour en savoir plus sur l'accès aux groupes et le filtrage par mention (et leur interaction), voir : [/channels/groups](/channels/groups).

## NickServ

Pour s'identifier auprès de NickServ après connexion :

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Enregistrement unique optionnel à la connexion :

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Désactivez `register` une fois le pseudonyme enregistré pour éviter les tentatives REGISTER répétées.

## Variables d'environnement

Le compte par défaut prend en charge :

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (séparés par des virgules)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Dépannage

- Si le bot se connecte mais ne répond jamais dans les canaux, vérifiez `channels.irc.groups` **et** si le filtrage par mention rejette les messages (`missing-mention`). Si vous voulez qu'il réponde sans pings, définissez `requireMention:false` pour le canal.
- Si la connexion échoue, vérifiez la disponibilité du pseudonyme et le mot de passe du serveur.
- Si TLS échoue sur un réseau personnalisé, vérifiez l'hôte/port et la configuration du certificat.
