---
summary: "Configuration, setup et utilisation du plugin LINE Messaging API"
read_when:
  - Vous voulez connecter OpenClaw a LINE
  - Vous avez besoin de la configuration webhook + identifiants LINE
  - Vous voulez les options de messages spécifiques a LINE
title: LINE
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/line.md"
  workflow: "manual"
---

# LINE (plugin)

LINE se connecte a OpenClaw via l'API LINE Messaging. Le plugin fonctionne comme un
recepteur de webhook sur le Gateway et utilise votre jeton d'accès canal + secret de canal pour
l'authentification.

Statut : pris en charge via plugin. Messages directs, discussions de groupe, média, localisations, messages
Flex, messages template et réponses rapides sont pris en charge. Les reactions et les fils
ne sont pas pris en charge.

## Plugin requis

Installez le plugin LINE :

```bash
openclaw plugins install @openclaw/line
```

Checkout local (depuis un depot git) :

```bash
openclaw plugins install ./extensions/line
```

## Configuration

1. Creez un compte LINE Developers et ouvrez la Console :
   [https://developers.line.biz/console/](https://developers.line.biz/console/)
2. Creez (ou selectionnez) un Provider et ajoutez un canal **Messaging API**.
3. Copiez le **Channel access token** et le **Channel secret** depuis les paramètres du canal.
4. Activez **Use webhook** dans les paramètres de l'API Messaging.
5. Définissez l'URL du webhook sur votre point de terminaison du Gateway (HTTPS requis) :

```
https://gateway-host/line/webhook
```

Le Gateway répond a la vérification du webhook LINE (GET) et aux événements entrants (POST).
Si vous avez besoin d'un chemin personnalisé, définissez `channels.line.webhookPath` ou
`channels.line.accounts.<id>.webhookPath` et mettez a jour l'URL en consequence.

## Configurer

Configuration minimale :

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

Variables d'environnement (compte par défaut uniquement) :

- `LINE_CHANNEL_ACCESS_TOKEN`
- `LINE_CHANNEL_SECRET`

Fichiers de jeton/secret :

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

Comptes multiples :

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## Contrôle d'accès

Les messages directs sont en mode appairage par défaut. Les expéditeurs inconnus reçoivent un code d'appairage et leurs
messages sont ignorés jusqu'a approbation.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Listes d'autorisation et politiques :

- `channels.line.dmPolicy` : `pairing | allowlist | open | disabled`
- `channels.line.allowFrom` : IDs utilisateur LINE autorisés pour les DM
- `channels.line.groupPolicy` : `allowlist | open | disabled`
- `channels.line.groupAllowFrom` : IDs utilisateur LINE autorisés pour les groupes
- Surcharges par groupe : `channels.line.groups.<groupId>.allowFrom`
- Note d'exécution : si `channels.line` est complètement absent, l'exécution se rabat sur `groupPolicy="allowlist"` pour les vérifications de groupe (même si `channels.defaults.groupPolicy` est défini).

Les IDs LINE sont sensibles a la casse. Les IDs valides ressemblent a :

- Utilisateur : `U` + 32 caractères hexadecimaux
- Groupe : `C` + 32 caractères hexadecimaux
- Salon : `R` + 32 caractères hexadecimaux

## Comportement des messages

- Le texte est decoupe a 5000 caractères.
- Le formatage Markdown est supprimé ; les blocs de code et tableaux sont convertis en cartes Flex
  quand possible.
- Les réponses en streaming sont mises en tampon ; LINE reçoit des morceaux complets avec une
  animation de chargement pendant que l'agent travaille.
- Les telechargements média sont limités par `channels.line.mediaMaxMb` (défaut 10).

## Donnees de canal (messages riches)

Utilisez `channelData.line` pour envoyer des réponses rapides, localisations, cartes Flex ou messages
template.

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Status card",
        contents: {
          /* Flex payload */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no",
      },
    },
  },
}
```

Le plugin LINE inclut aussi une commande `/card` pour les presets de messages Flex :

```
/card info "Welcome" "Thanks for joining!"
```

## Dépannage

- **Échec de la vérification du webhook :** assurez-vous que l'URL du webhook est en HTTPS et que le
  `channelSecret` correspond a la console LINE.
- **Pas d'événements entrants :** confirmez que le chemin du webhook correspond a `channels.line.webhookPath`
  et que le Gateway est joignable depuis LINE.
- **Erreurs de téléchargement média :** augmentez `channels.line.mediaMaxMb` si les média depassent la
  limité par défaut.
