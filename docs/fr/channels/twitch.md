---
summary: "Configuration et installation du bot de chat Twitch"
read_when:
  - Configuration de l'intégration chat Twitch pour OpenClaw
title: "Twitch"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/twitch.md
  workflow: manual
---

# Twitch (plugin)

Support du chat Twitch via connexion IRC. OpenClaw se connecte en tant qu'utilisateur Twitch (compte bot) pour recevoir et envoyer des messages dans les canaux.

## Plugin requis

Twitch est livré comme un plugin et n'est pas inclus dans l'installation de base.

Installation via CLI (registre npm) :

```bash
openclaw plugins install @openclaw/twitch
```

Checkout local (à partir d'un dépôt git) :

```bash
openclaw plugins install ./extensions/twitch
```

Détails : [Plugins](/tools/plugin)

## Configuration rapide (débutant)

1. Créez un compte Twitch dédié pour le bot (ou utilisez un compte existant).
2. Générez les identifiants : [Twitch Token Generator](https://twitchtokengenerator.com/)
   - Sélectionnez **Bot Token**
   - Vérifiez que les portées `chat:read` et `chat:write` sont sélectionnées
   - Copiez le **Client ID** et l'**Access Token**
3. Trouvez votre identifiant utilisateur Twitch : [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
4. Configurez le token :
   - Env : `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (compte par défaut uniquement)
   - Ou config : `channels.twitch.accessToken`
   - Si les deux sont définis, la configuration prend le dessus (l'env est un repli pour le compte par défaut uniquement).
5. Démarrez le Gateway.

**Important :** Ajoutez un contrôle d'accès (`allowFrom` ou `allowedRoles`) pour empêcher les utilisateurs non autorisés de déclencher le bot. `requireMention` est `true` par défaut.

Configuration minimale :

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // Compte Twitch du bot
      accessToken: "oauth:abc123...", // Token d'accès OAuth (ou utilisez la variable d'env OPENCLAW_TWITCH_ACCESS_TOKEN)
      clientId: "xyz789...", // Client ID du Token Generator
      channel: "vevisk", // Quel chat de canal Twitch rejoindre (requis)
      allowFrom: ["123456789"], // (recommandé) Votre identifiant utilisateur Twitch uniquement - obtenez-le sur https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    },
  },
}
```

## Description

- Un canal Twitch détenu par le Gateway.
- Routage déterministe : les réponses reviennent toujours vers Twitch.
- Chaque compte correspond à une clé de session isolée `agent:<agentId>:twitch:<accountName>`.
- `username` est le compte du bot (qui s'authentifié), `channel` est le salon de chat à rejoindre.

## Configuration (détaillée)

### Générer les identifiants

Utilisez [Twitch Token Generator](https://twitchtokengenerator.com/) :

- Sélectionnez **Bot Token**
- Vérifiez que les portées `chat:read` et `chat:write` sont sélectionnées
- Copiez le **Client ID** et l'**Access Token**

Pas d'inscription manuelle d'application nécessaire. Les tokens expirent après plusieurs heures.

### Configurer le bot

**Variable d'env (compte par défaut uniquement) :**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**Ou config :**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

Si les deux env et config sont définis, la configuration prend le dessus.

### Contrôle d'accès (recommandé)

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // (recommandé) Votre identifiant utilisateur Twitch uniquement
    },
  },
}
```

Préférez `allowFrom` pour une liste autorisée stricte. Utilisez plutôt `allowedRoles` si vous souhaitez un accès basé sur les rôles.

**Rôles disponibles :** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`.

**Pourquoi les identifiants utilisateur ?** Les noms d'utilisateur peuvent changer, permettant l'usurpation d'identité. Les identifiants utilisateur sont permanents.

Trouvez votre identifiant utilisateur Twitch : [https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/) (Convertir votre nom d'utilisateur Twitch en ID)

## Rafraîchissement de token (optionnel)

Les tokens de [Twitch Token Generator](https://twitchtokengenerator.com/) ne peuvent pas être automatiquement rafraîchis. Régénérez-les à expiration.

Pour le rafraîchissement automatique de token, créez votre propre application Twitch sur la [Console Développeur Twitch](https://dev.twitch.tv/console) et ajoutez à la config :

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```

Le bot rafraîchit automatiquement les tokens avant expiration et journalise les événements de rafraîchissement.

## Support multi-compte

Utilisez `channels.twitch.accounts` avec des tokens par compte. Voir [`gateway/configuration`](/gateway/configuration) pour le motif partagé.

Exemple (un compte bot dans deux canaux) :

```json5
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

**Note :** Chaque compte a besoin de son propre token (un token par canal).

## Contrôle d'accès

### Restrictions basées sur les rôles

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"],
        },
      },
    },
  },
}
```

### Liste autorisée par identifiant utilisateur (plus sécurisé)

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"],
        },
      },
    },
  },
}
```

### Accès basé sur les rôles (alternative)

`allowFrom` est une liste autorisée stricte. Lorsqu'elle est définie, seuls ces identifiants utilisateur sont autorisés.
Si vous souhaitez un accès basé sur les rôles, laissez `allowFrom` non défini et configurez `allowedRoles` à la place :

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

### Désactiver l'exigence de @mention

Par défaut, `requireMention` est `true`. Pour désactiver et répondre à tous les messages :

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false,
        },
      },
    },
  },
}
```

## Dépannage

Commencez par exécuter les commandes de diagnostic :

```bash
openclaw doctor
openclaw channels status --probe
```

### Le bot ne répond pas aux messages

**Vérifiez le contrôle d'accès :** Assurez-vous que votre identifiant utilisateur est dans `allowFrom`, ou supprimez temporairement
`allowFrom` et définissez `allowedRoles: ["all"]` pour tester.

**Vérifiez que le bot est dans le canal :** Le bot doit rejoindre le canal spécifie dans `channel`.

### Problèmes de token

**"Failed to connect" ou erreurs d'authentification :**

- Vérifiez que `accessToken` est la valeur du token d'accès OAuth (commence généralement par le préfixe `oauth:`)
- Vérifiez que le token a les portées `chat:read` et `chat:write`
- Si vous utilisez le rafraîchissement de token, vérifiez que `clientSecret` et `refreshToken` sont définis

### Le rafraîchissement de token ne fonctionne pas

**Consultez les logs pour les événements de rafraîchissement :**

```
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

Si vous voyez "token refresh disabled (no refresh token)" :

- Assurez-vous que `clientSecret` est fourni
- Assurez-vous que `refreshToken` est fourni

## Configuration

**Configuration de compte :**

- `username` - Nom d'utilisateur du bot
- `accessToken` - Token d'accès OAuth avec `chat:read` et `chat:write`
- `clientId` - Client ID Twitch (du Token Generator ou de votre application)
- `channel` - Canal à rejoindre (requis)
- `enabled` - Activer ce compte (par défaut : `true`)
- `clientSecret` - Optionnel : pour le rafraîchissement automatique de token
- `refreshToken` - Optionnel : pour le rafraîchissement automatique de token
- `expiresIn` - Expiration du token en secondes
- `obtainmentTimestamp` - Horodatage d'obtention du token
- `allowFrom` - Liste autorisée d'identifiants utilisateur
- `allowedRoles` - Contrôle d'accès basé sur les rôles (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
- `requireMention` - Exiger une @mention (par défaut : `true`)

**Options du fournisseur :**

- `channels.twitch.enabled` - Activer/désactiver le démarrage du canal
- `channels.twitch.username` - Nom d'utilisateur du bot (config simplifiée compte unique)
- `channels.twitch.accessToken` - Token d'accès OAuth (config simplifiée compte unique)
- `channels.twitch.clientId` - Client ID Twitch (config simplifiée compte unique)
- `channels.twitch.channel` - Canal à rejoindre (config simplifiée compte unique)
- `channels.twitch.accounts.<accountName>` - Config multi-compte (tous les champs de compte ci-dessus)

Exemple complet :

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

## Actions d'outil

L'agent peut appeler `twitch` avec l'action :

- `send` - Envoyer un message dans un canal

Exemple :

```json5
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```

## Sécurité et opérations

- **Traitez les tokens comme des mots de passe** - Ne commitez jamais de tokens dans git
- **Utilisez le rafraîchissement automatique de token** pour les bots longue durée
- **Utilisez les listes autorisées par identifiant utilisateur** au lieu des noms d'utilisateur pour le contrôle d'accès
- **Surveillez les logs** pour les événements de rafraîchissement de token et l'état de connexion
- **Limitez les portées des tokens** - Demandez uniquement `chat:read` et `chat:write`
- **En cas de blocage** : redémarrez le Gateway après avoir confirmé qu'aucun autre processus ne détient la session

## Limités

- **500 caractères** par message (découpage automatique aux limités de mots)
- Le Markdown est supprimé avant le découpage
- Pas de limitation de débit (utilisé les limités de débit intégrées de Twitch)
