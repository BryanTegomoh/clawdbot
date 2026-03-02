---
summary: "Configuration webhook Synology Chat et config OpenClaw"
read_when:
  - Configuration de Synology Chat avec OpenClaw
  - Debogage du routage webhook Synology Chat
title: "Synology Chat"
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/synology-chat.md"
  workflow: "manual"
---

# Synology Chat (plugin)

Statut : pris en charge via plugin en tant que canal de messages directs utilisant les webhooks Synology Chat.
Le plugin accepte les messages entrants depuis les webhooks sortants Synology Chat et envoie les réponses
via un webhook entrant Synology Chat.

## Plugin requis

Synology Chat est base sur un plugin et ne fait pas partie de l'installation par défaut des canaux.

Installation depuis un checkout local :

```bash
openclaw plugins install ./extensions/synology-chat
```

Details : [Plugins](/tools/plugin)

## Configuration rapide

1. Installez et activez le plugin Synology Chat.
2. Dans les integrations Synology Chat :
   - Creez un webhook entrant et copiez son URL.
   - Creez un webhook sortant avec votre jeton secret.
3. Pointez l'URL du webhook sortant vers votre Gateway OpenClaw :
   - `https://gateway-host/webhook/synology` par défaut.
   - Ou votre `channels.synology-chat.webhookPath` personnalisé.
4. Configurez `channels.synology-chat` dans OpenClaw.
5. Redemarrez le Gateway et envoyez un DM au bot Synology Chat.

Configuration minimale :

```json5
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## Variables d'environnement

Pour le compte par défaut, vous pouvez utiliser des variables d'environnement :

- `SYNOLOGY_CHAT_TOKEN`
- `SYNOLOGY_CHAT_INCOMING_URL`
- `SYNOLOGY_NAS_HOST`
- `SYNOLOGY_ALLOWED_USER_IDS` (séparées par des virgules)
- `SYNOLOGY_RATE_LIMIT`
- `OPENCLAW_BOT_NAME`

Les valeurs de configuration priment sur les variables d'environnement.

## Politique DM et contrôle d'accès

- `dmPolicy: "allowlist"` est la valeur par défaut recommandee.
- `allowedUserIds` accepte une liste (ou une chaine séparée par des virgules) d'IDs utilisateur Synology.
- En mode `allowlist`, une liste `allowedUserIds` vide est traitee comme une mauvaise configuration et la route webhook ne demarrera pas (utilisez `dmPolicy: "open"` pour autoriser tout le monde).
- `dmPolicy: "open"` autorisé tout expéditeur.
- `dmPolicy: "disabled"` bloque les DM.
- Les approbations d'appairage fonctionnent avec :
  - `openclaw pairing list synology-chat`
  - `openclaw pairing approve synology-chat <CODE>`

## Livraison sortante

Utilisez les IDs utilisateur numériques Synology Chat comme cibles.

Exemples :

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

L'envoi de média est pris en charge par la livraison de fichiers basee sur URL.

## Multi-comptes

Plusieurs comptes Synology Chat sont pris en charge sous `channels.synology-chat.accounts`.
Chaque compte peut surcharger le jeton, l'URL entrante, le chemin webhook, la politique DM et les limités.

```json5
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## Notes de sécurité

- Gardez le `token` secret et renouvelez-le en cas de fuite.
- Gardez `allowInsecureSsl: false` sauf si vous faites explicitement confiance a un certificat auto-signe de NAS local.
- Les requêtes webhook entrantes sont verifiees par jeton et limitées en debit par expéditeur.
- Préférez `dmPolicy: "allowlist"` pour la production.
