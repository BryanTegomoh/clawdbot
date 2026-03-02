---
title: Matrix
description: Connecter OpenClaw au protocole Matrix décentralisé.
summary: "Statut de prise en charge de Matrix, fonctionnalités et configuration"
read_when:
  - Vous travaillez sur les fonctionnalités du canal Matrix
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/matrix.md
  workflow: manual
---

# Matrix (plugin)

Matrix est un protocole de messagerie ouvert et décentralisé. OpenClaw se connecte en tant qu'**utilisateur** Matrix
sur n'importe quel serveur d'accueil, vous avez donc besoin d'un compte Matrix pour le bot. Une fois connecté, vous pouvez envoyer un Message privé
au bot directement ou l'inviter dans des salons (les « groupes » Matrix). Beeper est aussi une option de client valide,
mais il nécessite l'activation du E2EE.

Statut : pris en charge via plugin (@vector-im/matrix-bot-sdk). Messages privés, salons, fils, médias, réactions,
sondages (envoi + poll-start en texte), localisation et E2EE (avec support crypto).

## Plugin requis

Matrix est livré comme un plugin et n'est pas inclus dans l'installation de base.

Installation via CLI (registre npm) :

```bash
openclaw plugins install @openclaw/matrix
```

Checkout local (à partir d'un dépôt git) :

```bash
openclaw plugins install ./extensions/matrix
```

Si vous choisissez Matrix pendant la configuration/l'onboarding et qu'un checkout git est détecté,
OpenClaw proposera automatiquement le chemin d'installation local.

Détails : [Plugins](/tools/plugin)

## Configuration

1. Installez le plugin Matrix :
   - Depuis npm : `openclaw plugins install @openclaw/matrix`
   - Depuis un checkout local : `openclaw plugins install ./extensions/matrix`
2. Créez un compte Matrix sur un serveur d'accueil :
   - Parcourez les options d'hébergement sur [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
   - Ou hébergez-le vous-même.
3. Obtenez un token d'accès pour le compte bot :
   - Utilisez l'API de connexion Matrix avec `curl` sur votre serveur d'accueil :

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   - Remplacez `matrix.example.org` par l'URL de votre serveur d'accueil.
   - Ou définissez `channels.matrix.userId` + `channels.matrix.password` : OpenClaw appelle le même
     point de terminaison de connexion, stocke le token d'accès dans `~/.openclaw/credentials/matrix/credentials.json`,
     et le réutilise au prochain démarrage.

4. Configurez les identifiants :
   - Env : `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (ou `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
   - Ou config : `channels.matrix.*`
   - Si les deux sont définis, la configuration prend le dessus.
   - Avec un token d'accès : l'identifiant utilisateur est récupéré automatiquement via `/whoami`.
   - Lorsqu'il est défini, `channels.matrix.userId` devrait être l'identifiant Matrix complet (exemple : `@bot:example.org`).
5. Redémarrez le Gateway (ou terminez l'onboarding).
6. Démarrez un Message privé avec le bot ou invitez-le dans un salon depuis n'importe quel client Matrix
   (Élément, Beeper, etc. ; voir [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/)). Beeper nécessite le E2EE,
   donc définissez `channels.matrix.encryption: true` et vérifiez l'appareil.

Configuration minimale (token d'accès, identifiant utilisateur récupéré automatiquement) :

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

Configuration E2EE (chiffrement de bout en bout activé) :

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## Chiffrement (E2EE)

Le chiffrement de bout en bout est **pris en charge** via le SDK crypto Rust.

Activez avec `channels.matrix.encryption: true` :

- Si le module crypto se charge, les salons chiffrés sont déchiffrés automatiquement.
- Les médias sortants sont chiffrés lors de l'envoi vers des salons chiffrés.
- À la première connexion, OpenClaw demande la vérification de l'appareil depuis vos autres sessions.
- Vérifiez l'appareil dans un autre client Matrix (Élément, etc.) pour activer le partage de clés.
- Si le module crypto ne peut pas être chargé, le E2EE est désactivé et les salons chiffrés ne seront pas déchiffrés ;
  OpenClaw journalise un avertissement.
- Si vous voyez des erreurs de module crypto manquant (par exemple, `@matrix-org/matrix-sdk-crypto-nodejs-*`),
  autorisez les scripts de build pour `@matrix-org/matrix-sdk-crypto-nodejs` et exécutez
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` ou récupérez le binaire avec
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

L'état crypto est stocké par compte + token d'accès dans
`~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
(base de données SQLite). L'état de synchronisation réside à côté dans `bot-storage.json`.
Si le token d'accès (appareil) change, un nouveau magasin est créé et le bot doit être
re-vérifié pour les salons chiffrés.

**Vérification de l'appareil :**
Lorsque le E2EE est activé, le bot demandera la vérification depuis vos autres sessions au démarrage.
Ouvrez Élément (ou un autre client) et approuvez la demande de vérification pour établir la confiance.
Une fois vérifié, le bot peut déchiffrer les messages dans les salons chiffrés.

## Multi-compte

Support multi-compte : utilisez `channels.matrix.accounts` avec des identifiants par compte et un `name` optionnel. Voir [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le motif partagé.

Chaque compte fonctionne comme un utilisateur Matrix séparé sur n'importe quel serveur d'accueil. La configuration par compte
hérite des paramètres de niveau supérieur `channels.matrix` et peut remplacer n'importe quelle option
(politique MP, groupes, chiffrement, etc.).

```json5
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "Main assistant",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "Alerts bot",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

Notes :

- Le démarrage des comptes est sérialisé pour éviter les conditions de concurrence avec les imports de modules simultanés.
- Les variables d'environnement (`MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`, etc.) ne s'appliquent qu'au compte **par défaut**.
- Les paramètres de canal de base (politique MP, politique de groupe, filtrage par mention, etc.) s'appliquent à tous les comptes sauf remplacement par compte.
- Utilisez `bindings[].match.accountId` pour router chaque compte vers un agent différent.
- L'état crypto est stocké par compte + token d'accès (magasins de clés séparés par compte).

## Modèle de routage

- Les réponses reviennent toujours vers Matrix.
- Les Messages privés partagent la session principale de l'agent ; les salons correspondent à des sessions de groupe.

## Contrôle d'accès (Messages privés)

- Par défaut : `channels.matrix.dm.policy = "pairing"`. Les expéditeurs inconnus reçoivent un code d'appairage.
- Approbation via :
  - `openclaw pairing list matrix`
  - `openclaw pairing approve matrix <CODE>`
- Messages privés publics : `channels.matrix.dm.policy="open"` plus `channels.matrix.dm.allowFrom=["*"]`.
- `channels.matrix.dm.allowFrom` accepte les identifiants utilisateur Matrix complets (exemple : `@user:server`). L'assistant résout les noms d'affichage en identifiants utilisateur lorsque la recherche dans l'annuaire trouve une correspondance exacte unique.
- N'utilisez pas les noms d'affichage ou les localparts nus (exemple : `"Alice"` ou `"alice"`). Ils sont ambigus et ignorés pour la correspondance de liste autorisée. Utilisez les identifiants complets `@user:server`.

## Salons (groupes)

- Par défaut : `channels.matrix.groupPolicy = "allowlist"` (filtrage par mention). Utilisez `channels.defaults.groupPolicy` pour remplacer la valeur par défaut lorsqu'elle n'est pas définie.
- Note d'exécution : si `channels.matrix` est complètement absent, l'exécution revient à `groupPolicy="allowlist"` pour les vérifications de salon (même si `channels.defaults.groupPolicy` est défini).
- Liste autorisée des salons avec `channels.matrix.groups` (identifiants de salon ou alias ; les noms sont résolus en identifiants lorsque la recherche dans l'annuaire trouve une correspondance exacte unique) :

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

- `requireMention: false` activé la réponse automatique dans ce salon.
- `groups."*"` peut définir les valeurs par défaut pour le filtrage par mention dans tous les salons.
- `groupAllowFrom` restreint quels expéditeurs peuvent déclencher le bot dans les salons (identifiants utilisateur Matrix complets).
- Les listes autorisées `users` par salon peuvent restreindre davantage les expéditeurs dans un salon spécifique (utilisez les identifiants utilisateur Matrix complets).
- L'assistant de configuration invite à configurer les listes autorisées de salons (identifiants de salon, alias ou noms) et ne résout les noms que sur une correspondance exacte et unique.
- Au démarrage, OpenClaw résout les noms de salons/utilisateurs dans les listes autorisées en identifiants et journalise la correspondance ; les entrées non résolues sont ignorées pour la correspondance de liste autorisée.
- Les invitations sont acceptées automatiquement par défaut ; contrôlez avec `channels.matrix.autoJoin` et `channels.matrix.autoJoinAllowlist`.
- Pour n'autoriser **aucun salon**, définissez `channels.matrix.groupPolicy: "disabled"` (ou gardez une liste autorisée vide).
- Clé héritée : `channels.matrix.rooms` (même structure que `groups`).

## Fils

- Le threading de réponses est pris en charge.
- `channels.matrix.threadReplies` contrôle si les réponses restent dans les fils :
  - `off`, `inbound` (par défaut), `always`
- `channels.matrix.replyToMode` contrôle les métadonnées de réponse lorsqu'il ne répond pas dans un fil :
  - `off` (par défaut), `first`, `all`

## Fonctionnalités

| Fonctionnalité  | Statut                                                                                  |
| --------------- | --------------------------------------------------------------------------------------- |
| Messages privés | Pris en charge                                                                          |
| Salons          | Pris en charge                                                                          |
| Fils            | Pris en charge                                                                          |
| Médias          | Pris en charge                                                                          |
| E2EE            | Pris en charge (module crypto requis)                                                   |
| Réactions       | Pris en charge (envoi/lecture via outils)                                                |
| Sondages        | Envoi pris en charge ; les démarrages de sondage entrants sont convertis en texte (réponses/fins ignorées) |
| Localisation    | Pris en charge (URI géo ; altitude ignorée)                                             |
| Commandes natives | Pris en charge                                                                        |

## Dépannage

Suivez d'abord cette procédure :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Puis confirmez l'état d'appairage MP si nécessaire :

```bash
openclaw pairing list matrix
```

Échecs courants :

- Connecté mais les messages de salon sont ignorés : salon bloque par `groupPolicy` ou la liste autorisée de salon.
- Messages privés ignorés : expéditeur en attente d'approbation lorsque `channels.matrix.dm.policy="pairing"`.
- Les salons chiffrés échouent : support crypto ou paramètres de chiffrement incompatibles.

Pour le flux de triage : [/channels/troubleshooting](/channels/troubleshooting).

## Référence de configuration (Matrix)

Configuration complète : [Configuration](/gateway/configuration)

Options du fournisseur :

- `channels.matrix.enabled` : activer/désactiver le démarrage du canal.
- `channels.matrix.homeserver` : URL du serveur d'accueil.
- `channels.matrix.userId` : identifiant utilisateur Matrix (optionnel avec token d'accès).
- `channels.matrix.accessToken` : token d'accès.
- `channels.matrix.password` : mot de passe pour la connexion (token stocké).
- `channels.matrix.deviceName` : nom d'affichage de l'appareil.
- `channels.matrix.encryption` : activer le E2EE (par défaut : false).
- `channels.matrix.initialSyncLimit` : limité de synchronisation initiale.
- `channels.matrix.threadReplies` : `off | inbound | always` (par défaut : inbound).
- `channels.matrix.textChunkLimit` : taille de découpe du texte sortant (caractères).
- `channels.matrix.chunkMode` : `length` (par défaut) ou `newline` pour découper sur les lignes vides (limités de paragraphe) avant la découpe par longueur.
- `channels.matrix.dm.policy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
- `channels.matrix.dm.allowFrom` : liste autorisée MP (identifiants utilisateur Matrix complets). `open` nécessite `"*"`. L'assistant résout les noms en identifiants lorsque possible.
- `channels.matrix.groupPolicy` : `allowlist | open | disabled` (par défaut : allowlist).
- `channels.matrix.groupAllowFrom` : expéditeurs autorisés pour les messages de groupe (identifiants utilisateur Matrix complets).
- `channels.matrix.allowlistOnly` : forcer les règles de liste autorisée pour les MP + salons.
- `channels.matrix.groups` : liste autorisée de groupe + carte des paramètres par salon.
- `channels.matrix.rooms` : liste autorisée/configuration de groupe héritée.
- `channels.matrix.replyToMode` : mode de réponse pour les fils/tags.
- `channels.matrix.mediaMaxMb` : limité de média entrant/sortant (Mo).
- `channels.matrix.autoJoin` : gestion des invitations (`always | allowlist | off`, par défaut : always).
- `channels.matrix.autoJoinAllowlist` : identifiants/alias de salon autorisés pour l'acceptation automatique.
- `channels.matrix.accounts` : configuration multi-compte indexée par identifiant de compte (chaque compte hérite des paramètres de niveau supérieur).
- `channels.matrix.actions` : filtrage d'outils par action (réactions/messages/épinglages/memberInfo/channelInfo).
