---
summary: "Statut du support Nextcloud Talk, capacités et configuration"
read_when:
  - Travail sur les fonctionnalités du canal Nextcloud Talk
title: "Nextcloud Talk"
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/nextcloud-talk.md"
  workflow: "manual"
---

# Nextcloud Talk (plugin)

Statut : pris en charge via plugin (bot webhook). Messages directs, salons, reactions et messages markdown sont pris en charge.

## Plugin requis

Nextcloud Talk est distribue en tant que plugin et n'est pas inclus dans l'installation de base.

Installation via CLI (registre npm) :

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Checkout local (depuis un depot git) :

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Si vous choisissez Nextcloud Talk pendant la configuration/configuration initiale et qu'un checkout git est détecté,
OpenClaw proposera automatiquement le chemin d'installation local.

Details : [Plugins](/tools/plugin)

## Configuration rapide (debutant)

1. Installez le plugin Nextcloud Talk.
2. Sur votre serveur Nextcloud, creez un bot :

   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```

3. Activez le bot dans les paramètres du salon cible.
4. Configurez OpenClaw :
   - Config : `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   - Ou env : `NEXTCLOUD_TALK_BOT_SECRET` (compte par défaut uniquement)
5. Redemarrez le Gateway (ou terminez la configuration initiale).

Configuration minimale :

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## Notes

- Les bots ne peuvent pas initier de DM. L'utilisateur doit envoyer un message au bot en premier.
- L'URL du webhook doit être joignable par le Gateway ; définissez `webhookPublicUrl` si derriere un proxy.
- Les telechargements média ne sont pas pris en charge par l'API bot ; les média sont envoyés sous forme d'URLs.
- Le payload du webhook ne distingue pas les DM des salons ; définissez `apiUser` + `apiPassword` pour activer les recherches de type de salon (sinon les DM sont traites comme des salons).

## Contrôle d'accès (DM)

- Par défaut : `channels.nextcloud-talk.dmPolicy = "pairing"`. Les expéditeurs inconnus reçoivent un code d'appairage.
- Approbation via :
  - `openclaw pairing list nextcloud-talk`
  - `openclaw pairing approve nextcloud-talk <CODE>`
- DM publics : `channels.nextcloud-talk.dmPolicy="open"` plus `channels.nextcloud-talk.allowFrom=["*"]`.
- `allowFrom` correspond uniquement aux IDs utilisateur Nextcloud ; les noms d'affichage sont ignorés.

## Salons (groupes)

- Par défaut : `channels.nextcloud-talk.groupPolicy = "allowlist"` (avec déclenchement par mention).
- Liste d'autorisation des salons avec `channels.nextcloud-talk.rooms` :

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

- Pour n'autoriser aucun salon, gardez la liste d'autorisation vide ou définissez `channels.nextcloud-talk.groupPolicy="disabled"`.

## Capacités

| Fonctionnalité   | Statut          |
| ----------------- | --------------- |
| Messages directs  | Pris en charge  |
| Salons            | Pris en charge  |
| Fils              | Non pris en charge |
| Média             | URL uniquement  |
| Réactions         | Pris en charge  |
| Commandes natives | Non pris en charge |

## Référence de configuration (Nextcloud Talk)

Configuration complète : [Configuration](/gateway/configuration)

Options du fournisseur :

- `channels.nextcloud-talk.enabled` : activer/désactiver le démarrage du canal.
- `channels.nextcloud-talk.baseUrl` : URL de l'instance Nextcloud.
- `channels.nextcloud-talk.botSecret` : secret partage du bot.
- `channels.nextcloud-talk.botSecretFile` : chemin du fichier secret.
- `channels.nextcloud-talk.apiUser` : utilisateur API pour les recherches de salon (détection DM).
- `channels.nextcloud-talk.apiPassword` : mot de passe API/app pour les recherches de salon.
- `channels.nextcloud-talk.apiPasswordFile` : chemin du fichier de mot de passe API.
- `channels.nextcloud-talk.webhookPort` : port d'écoute du webhook (défaut : 8788).
- `channels.nextcloud-talk.webhookHost` : hôte du webhook (défaut : 0.0.0.0).
- `channels.nextcloud-talk.webhookPath` : chemin du webhook (défaut : /nextcloud-talk-webhook).
- `channels.nextcloud-talk.webhookPublicUrl` : URL du webhook joignable de l'exterieur.
- `channels.nextcloud-talk.dmPolicy` : `pairing | allowlist | open | disabled`.
- `channels.nextcloud-talk.allowFrom` : liste d'autorisation DM (IDs utilisateur). `open` nécessite `"*"`.
- `channels.nextcloud-talk.groupPolicy` : `allowlist | open | disabled`.
- `channels.nextcloud-talk.groupAllowFrom` : liste d'autorisation de groupe (IDs utilisateur).
- `channels.nextcloud-talk.rooms` : paramètres par salon et liste d'autorisation.
- `channels.nextcloud-talk.historyLimit` : limité d'historique de groupe (0 désactive).
- `channels.nextcloud-talk.dmHistoryLimit` : limité d'historique DM (0 désactive).
- `channels.nextcloud-talk.dms` : surcharges par DM (historyLimit).
- `channels.nextcloud-talk.textChunkLimit` : taille de decoupe du texte sortant (caractères).
- `channels.nextcloud-talk.chunkMode` : `length` (défaut) ou `newline` pour decouper sur les lignes vides (limités de paragraphe) avant la decoupe par longueur.
- `channels.nextcloud-talk.blockStreaming` : désactiver le streaming de blocs pour ce canal.
- `channels.nextcloud-talk.blockStreamingCoalesce` : reglage de la coalescence du streaming de blocs.
- `channels.nextcloud-talk.mediaMaxMb` : limité média entrant (Mo).
