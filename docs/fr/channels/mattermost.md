---
title: Mattermost
description: Connecter OpenClaw à Mattermost via un bot et WebSocket.
summary: "Configuration du bot Mattermost et intégration OpenClaw"
read_when:
  - Vous configurez Mattermost
  - Vous déboguez le routage Mattermost
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/mattermost.md
  workflow: manual
---

# Mattermost (plugin)

Statut : pris en charge via plugin (token de bot + événements WebSocket). Les canaux, groupes et Messages privés sont pris en charge.
Mattermost est une plateforme de messagerie d'équipe auto-hébergeable ; consultez le site officiel sur
[mattermost.com](https://mattermost.com) pour les détails du produit et les téléchargements.

## Plugin requis

Mattermost est livré comme un plugin et n'est pas inclus dans l'installation de base.

Installation via CLI (registre npm) :

```bash
openclaw plugins install @openclaw/mattermost
```

Checkout local (à partir d'un dépôt git) :

```bash
openclaw plugins install ./extensions/mattermost
```

Si vous choisissez Mattermost pendant la configuration/l'onboarding et qu'un checkout git est détecté,
OpenClaw proposera automatiquement le chemin d'installation local.

Détails : [Plugins](/tools/plugin)

## Configuration rapide

1. Installez le plugin Mattermost.
2. Créez un compte bot Mattermost et copiez le **token de bot**.
3. Copiez l'**URL de base** Mattermost (par ex., `https://chat.example.com`).
4. Configurez OpenClaw et démarrez le Gateway.

Configuration minimale :

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## Variables d'environnement (compte par défaut)

Définissez-les sur l'hôte Gateway si vous préférez les variables d'environnement :

- `MATTERMOST_BOT_TOKEN=...`
- `MATTERMOST_URL=https://chat.example.com`

Les variables d'environnement s'appliquent uniquement au compte **par défaut** (`default`). Les autres comptes doivent utiliser des valeurs de configuration.

## Modes de chat

Mattermost répond automatiquement aux Messages privés. Le comportement des canaux est contrôlé par `chatmode` :

- `oncall` (par défaut) : ne répond que lorsqu'il est @mentionné dans les canaux.
- `onmessage` : répond à chaque message du canal.
- `onchar` : répond lorsqu'un message commence par un préfixe déclencheur.

Exemple de configuration :

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

Notes :

- `onchar` répond toujours aux @mentions explicites.
- `channels.mattermost.requireMention` est respecté pour les configurations héritées mais `chatmode` est préféré.

## Contrôle d'accès (Messages privés)

- Par défaut : `channels.mattermost.dmPolicy = "pairing"` (les expéditeurs inconnus reçoivent un code d'appairage).
- Approbation via :
  - `openclaw pairing list mattermost`
  - `openclaw pairing approve mattermost <CODE>`
- Messages privés publics : `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]`.

## Canaux (groupes)

- Par défaut : `channels.mattermost.groupPolicy = "allowlist"` (filtrage par mention).
- Liste autorisée des expéditeurs avec `channels.mattermost.groupAllowFrom` (identifiants utilisateur recommandés).
- La correspondance par `@username` est mutable et n'est activée que lorsque `channels.mattermost.dangerouslyAllowNameMatching: true`.
- Canaux ouverts : `channels.mattermost.groupPolicy="open"` (filtrage par mention).
- Note d'exécution : si `channels.mattermost` est complètement absent, l'exécution revient à `groupPolicy="allowlist"` pour les vérifications de groupe (même si `channels.defaults.groupPolicy` est défini).

## Cibles pour la livraison sortante

Utilisez ces formats de cible avec `openclaw message send` ou cron/webhooks :

- `channel:<id>` pour un canal
- `user:<id>` pour un Message privé
- `@username` pour un Message privé (résolu via l'API Mattermost)

Les identifiants nus sont traités comme des canaux.

## Réactions (outil message)

- Utilisez `message action=react` avec `channel=mattermost`.
- `messageId` est l'identifiant de post Mattermost.
- `emoji` accepte des noms comme `thumbsup` ou `:+1:` (les deux-points sont optionnels).
- Définissez `remove=true` (booléen) pour supprimer une réaction.
- Les événements d'ajout/suppression de réaction sont transmis en tant qu'événements système à la session de l'agent route.

Exemples :

```
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

Configuration :

- `channels.mattermost.actions.reactions` : activer/désactiver les actions de réaction (par défaut true).
- Remplacement par compte : `channels.mattermost.accounts.<id>.actions.reactions`.

## Multi-compte

Mattermost prend en charge plusieurs comptes sous `channels.mattermost.accounts` :

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## Dépannage

- Pas de réponses dans les canaux : assurez-vous que le bot est dans le canal et mentionnez-le (oncall), utilisez un préfixe déclencheur (onchar), ou définissez `chatmode: "onmessage"`.
- Erreurs d'authentification : vérifiez le token de bot, l'URL de base et si le compte est activé.
- Problèmes multi-compte : les variables d'environnement ne s'appliquent qu'au compte `default`.
