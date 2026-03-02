---
summary: "Statut du support Tlon/Urbit, capacités et configuration"
read_when:
  - Travail sur les fonctionnalités du canal Tlon/Urbit
title: "Tlon"
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/tlon.md"
  workflow: "manual"
---

# Tlon (plugin)

Tlon est un messager decentralise construit sur Urbit. OpenClaw se connecte a votre vaisseau Urbit et peut
répondre aux DM et aux messages de discussion de groupe. Les réponses de groupe nécessitent une @mention par défaut et peuvent
être davantage restreintes via des listes d'autorisation.

Statut : pris en charge via plugin. DM, mentions de groupe, réponses en fil, et repli média texte seul
(URL ajoutée a la légende). Les reactions, sondages et telechargements média natifs ne sont pas pris en charge.

## Plugin requis

Tlon est distribue en tant que plugin et n'est pas inclus dans l'installation de base.

Installation via CLI (registre npm) :

```bash
openclaw plugins install @openclaw/tlon
```

Checkout local (depuis un depot git) :

```bash
openclaw plugins install ./extensions/tlon
```

Details : [Plugins](/tools/plugin)

## Configuration

1. Installez le plugin Tlon.
2. Rassemblez l'URL de votre vaisseau et le code de connexion.
3. Configurez `channels.tlon`.
4. Redemarrez le Gateway.
5. Envoyez un DM au bot ou mentionnez-le dans un canal de groupe.

Configuration minimale (compte unique) :

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
    },
  },
}
```

URLs de vaisseau privé/LAN (avance) :

Par défaut, OpenClaw bloque les noms d'hôte et plages d'IP privés/internes pour ce plugin (durcissement SSRF).
Si l'URL de votre vaisseau est sur un réseau privé (par exemple `http://192.168.1.50:8080` ou `http://localhost:8080`),
vous devez opter explicitement :

```json5
{
  channels: {
    tlon: {
      allowPrivateNetwork: true,
    },
  },
}
```

## Canaux de groupe

La decouverte automatique est activée par défaut. Vous pouvez aussi epingler des canaux manuellement :

```json5
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

Désactiver la decouverte automatique :

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## Contrôle d'accès

Liste d'autorisation DM (vide = autoriser tous) :

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

Autorisation de groupe (restreint par défaut) :

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## Cibles de livraison (CLI/cron)

Utilisez-les avec `openclaw message send` ou la livraison cron :

- DM : `~sampel-palnet` ou `dm/~sampel-palnet`
- Groupe : `chat/~host-ship/channel` ou `group:~host-ship/channel`

## Notes

- Les réponses de groupe nécessitent une mention (par ex. `~your-bot-ship`) pour répondre.
- Réponses en fil : si le message entrant est dans un fil, OpenClaw répond dans le fil.
- Média : `sendMedia` se rabat sur texte + URL (pas de téléchargement natif).
