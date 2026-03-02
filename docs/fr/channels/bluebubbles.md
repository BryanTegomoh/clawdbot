---
summary: "iMessage via le serveur macOS BlueBubbles (envoi/reception REST, saisie, reactions, appairage, actions avancees)."
read_when:
  - Configuration du canal BlueBubbles
  - Dépannage de l'appairage webhook
  - Configuration d'iMessage sur macOS
title: "BlueBubbles"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/bluebubbles.md
  workflow: manual
---

# BlueBubbles (macOS REST)

Statut : plugin intégré qui communique avec le serveur macOS BlueBubbles via HTTP. **Recommandé pour l'intégration iMessage** grace a son API plus riche et sa configuration plus facile par rapport au canal imsg ancien.

## Aperçu

- Fonctionne sur macOS via l'application helper BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
- Recommandé/teste : macOS Sequoia (15). macOS Tahoe (26) fonctionne ; la modification est actuellement cassee sur Tahoe, et les mises a jour d'icone de groupe peuvent signaler un succes mais ne pas se synchroniser.
- OpenClaw communique avec lui via son API REST (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
- Les messages entrants arrivent via les webhooks ; les réponses sortantes, les indicateurs de saisie, les accuses de reception et les tapbacks sont des appels REST.
- Les pieces jointes et les stickers sont ingeres comme médias entrants (et présentés a l'agent lorsque c'est possible).
- L'appairage/liste autorisée fonctionne de la même maniere que les autres canaux (`/channels/pairing` etc) avec `channels.bluebubbles.allowFrom` + codes d'appairage.
- Les reactions sont presentees comme des événements système tout comme Slack/Telegram pour que les agents puissent les « mentionner » avant de répondre.
- Fonctionnalités avancees : modification, annulation d'envoi, fil de réponse, effets de message, gestion de groupe.

## Démarrage rapide

1. Installez le serveur BlueBubbles sur votre Mac (suivez les instructions sur [bluebubbles.app/install](https://bluebubbles.app/install)).
2. Dans la configuration BlueBubbles, activez l'API web et définissez un mot de passe.
3. Exécutez `openclaw onboard` et selectionnez BlueBubbles, ou configurez manuellement :

   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook",
       },
     },
   }
   ```

4. Pointez les webhooks BlueBubbles vers votre gateway (exemple : `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`).
5. Démarrez le gateway ; il enregistrera le gestionnaire de webhook et commencera l'appairage.

Note de sécurité :

- Définissez toujours un mot de passe de webhook.
- L'authentification du webhook est toujours requise. OpenClaw rejette les requêtes webhook BlueBubbles sauf si elles incluent un mot de passe/guid qui correspond a `channels.bluebubbles.password` (par exemple `?password=<password>` ou `x-password`), quelle que soit la topologie loopback/proxy.

## Garder Messages.app actif (VM / configurations headless)

Certaines configurations de VM macOS / toujours allumees peuvent se retrouver avec Messages.app en « veille » (les événements entrants s'arretent jusqu'a ce que l'application soit ouverte/mise au premier plan). Une solution simple est de **solliciter Messages toutes les 5 minutes** avec un AppleScript + LaunchAgent.

### 1) Sauvegarder l'AppleScript

Sauvegardez ceci sous :

- `~/Scripts/poke-messages.scpt`

Exemple de script (non interactif ; ne vole pas le focus) :

```applescript
try
  tell application "Messages"
    if not running then
      launch
    end if

    -- Touch the scripting interface to keep the process responsive.
    set _chatCount to (count of chats)
  end tell
on error
  -- Ignore transient failures (first-run prompts, locked session, etc).
end try
```

### 2) Installer un LaunchAgent

Sauvegardez ceci sous :

- `~/Library/LaunchAgents/com.user.poke-messages.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

Notes :

- Ceci s'exécute **toutes les 300 secondes** et **a la connexion**.
- La première exécution peut déclencher des invites macOS **d'automatisation** (`osascript` -> Messages). Approuvez-les dans la même session utilisateur que celle qui exécute le LaunchAgent.

Chargez-le :

```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## Intégration

BlueBubbles est disponible dans l'assistant de configuration interactif :

```
openclaw onboard
```

L'assistant demande :

- **URL du serveur** (requis) : adresse du serveur BlueBubbles (par ex., `http://192.168.1.100:1234`)
- **Mot de passe** (requis) : mot de passe API depuis les paramètres du serveur BlueBubbles
- **Chemin du webhook** (optionnel) : par défaut `/bluebubbles-webhook`
- **Politique de messages privés** : pairing, allowlist, open ou disabled
- **Liste autorisée** : numéros de téléphone, emails ou cibles de discussion

Vous pouvez aussi ajouter BlueBubbles via CLI :

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## Contrôle d'accès (messages privés + groupes)

Messages privés :

- Par défaut : `channels.bluebubbles.dmPolicy = "pairing"`.
- Les expéditeurs inconnus reçoivent un code d'appairage ; les messages sont ignorés jusqu'a approbation (les codes expirent après 1 heure).
- Approuver via :
  - `openclaw pairing list bluebubbles`
  - `openclaw pairing approve bluebubbles <CODE>`
- L'appairage est l'echange de jetons par défaut. Details : [Appairage](/channels/pairing)

Groupes :

- `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (par défaut : `allowlist`).
- `channels.bluebubbles.groupAllowFrom` contrôle qui peut déclencher dans les groupes lorsque `allowlist` est défini.

### Filtrage par mention (groupes)

BlueBubbles prend en charge le filtrage par mention pour les discussions de groupe, correspondant au comportement iMessage/WhatsApp :

- Utilisé `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`) pour détecter les mentions.
- Lorsque `requireMention` est activé pour un groupe, l'agent ne répond que lorsqu'il est mentionne.
- Les commandes de contrôle des expéditeurs autorisés contournent le filtrage par mention.

Configuration par groupe :

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // par défaut pour tous les groupes
        "iMessage;-;chat123": { requireMention: false }, // remplacement pour un groupe spécifique
      },
    },
  },
}
```

### Filtrage des commandes

- Les commandes de contrôle (par ex., `/config`, `/model`) nécessitent une autorisation.
- Utilisé `allowFrom` et `groupAllowFrom` pour déterminer l'autorisation des commandes.
- Les expéditeurs autorisés peuvent exécuter des commandes de contrôle même sans mentionner dans les groupes.

## Saisie + accuses de reception

- **Indicateurs de saisie** : envoyés automatiquement avant et pendant la génération de la réponse.
- **Accuses de reception** : contrôles par `channels.bluebubbles.sendReadReceipts` (par défaut : `true`).
- **Indicateurs de saisie** : OpenClaw envoie des événements de debut de saisie ; BlueBubbles efface automatiquement la saisie a l'envoi ou au timeout (l'arrêt manuel via DELETE n'est pas fiable).

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // desactiver les accuses de reception
    },
  },
}
```

## Actions avancees

BlueBubbles prend en charge les actions de message avancees lorsqu'elles sont activées dans la configuration :

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // tapbacks (par defaut : true)
        edit: true, // modifier les messages envoyes (macOS 13+, casse sur macOS 26 Tahoe)
        unsend: true, // annuler l'envoi de messages (macOS 13+)
        reply: true, // fil de réponse par GUID de message
        sendWithEffect: true, // effets de message (slam, loud, etc.)
        renameGroup: true, // renommer les discussions de groupe
        setGroupIcon: true, // definir l'icone/photo de la discussion de groupe (instable sur macOS 26 Tahoe)
        addParticipant: true, // ajouter des participants aux groupes
        removeParticipant: true, // retirer des participants des groupes
        leaveGroup: true, // quitter les discussions de groupe
        sendAttachment: true, // envoyer des pieces jointes/medias
      },
    },
  },
}
```

Actions disponibles :

- **react** : ajouter/supprimer des reactions tapback (`messageId`, `emoji`, `remove`)
- **edit** : modifier un message envoyé (`messageId`, `text`)
- **unsend** : annuler l'envoi d'un message (`messageId`)
- **reply** : répondre a un message spécifique (`messageId`, `text`, `to`)
- **sendWithEffect** : envoyer avec un effet iMessage (`text`, `to`, `effectId`)
- **renameGroup** : renommer une discussion de groupe (`chatGuid`, `displayName`)
- **setGroupIcon** : définir l'icone/photo d'une discussion de groupe (`chatGuid`, `média`) : instable sur macOS 26 Tahoe (l'API peut retourner un succes mais l'icone ne se synchronise pas).
- **addParticipant** : ajouter quelqu'un a un groupe (`chatGuid`, `address`)
- **removeParticipant** : retirer quelqu'un d'un groupe (`chatGuid`, `address`)
- **leaveGroup** : quitter une discussion de groupe (`chatGuid`)
- **sendAttachment** : envoyer des médias/fichiers (`to`, `buffer`, `filename`, `asVoice`)
  - Messages vocaux : définissez `asVoice: true` avec un audio **MP3** ou **CAF** pour envoyer comme message vocal iMessage. BlueBubbles convertit MP3 en CAF lors de l'envoi de messages vocaux.

### Identifiants de message (court vs complet)

OpenClaw peut présenter des identifiants de message _courts_ (par ex., `1`, `2`) pour economiser des tokens.

- `MessageSid` / `ReplyToId` peuvent être des identifiants courts.
- `MessageSidFull` / `ReplyToIdFull` contiennent les identifiants complets du fournisseur.
- Les identifiants courts sont en mémoire ; ils peuvent expirer au redémarrage ou a l'eviction du cache.
- Les actions acceptent un `messageId` court ou complet, mais les identifiants courts donneront une erreur s'ils ne sont plus disponibles.

Utilisez les identifiants complets pour les automatisations et le stockage durables :

- Modèles : `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
- Contexte : `MessageSidFull` / `ReplyToIdFull` dans les payloads entrants

Voir [Configuration](/gateway/configuration) pour les variables de modèle.

## Diffusion par blocs

Controllez si les réponses sont envoyées comme un seul message ou diffusees par blocs :

```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true, // activer la diffusion par blocs (desactive par defaut)
    },
  },
}
```

## Médias + limités

- Les pieces jointes entrantes sont telechargees et stockees dans le cache média.
- Limité média via `channels.bluebubbles.mediaMaxMb` (par défaut : 8 Mo).
- Le texte sortant est decoupe a `channels.bluebubbles.textChunkLimit` (par défaut : 4000 caractères).

## Référence de configuration

Configuration complète : [Configuration](/gateway/configuration)

Options du fournisseur :

- `channels.bluebubbles.enabled` : activer/désactiver le canal.
- `channels.bluebubbles.serverUrl` : URL de base de l'API REST BlueBubbles.
- `channels.bluebubbles.password` : mot de passe de l'API.
- `channels.bluebubbles.webhookPath` : chemin du point de terminaison webhook (par défaut : `/bluebubbles-webhook`).
- `channels.bluebubbles.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : `pairing`).
- `channels.bluebubbles.allowFrom` : liste autorisée de messages privés (identifiants, emails, numéros E.164, `chat_id:*`, `chat_guid:*`).
- `channels.bluebubbles.groupPolicy` : `open | allowlist | disabled` (par défaut : `allowlist`).
- `channels.bluebubbles.groupAllowFrom` : liste autorisée d'expéditeurs de groupe.
- `channels.bluebubbles.groups` : configuration par groupe (`requireMention`, etc.).
- `channels.bluebubbles.sendReadReceipts` : envoyer des accuses de reception (par défaut : `true`).
- `channels.bluebubbles.blockStreaming` : activer la diffusion par blocs (par défaut : `false` ; nécessaire pour les réponses en streaming).
- `channels.bluebubbles.textChunkLimit` : taille de découpage sortant en caractères (par défaut : 4000).
- `channels.bluebubbles.chunkMode` : `length` (par défaut) decoupe uniquement lorsque la limité `textChunkLimit` est dépassée ; `newline` decoupe sur les lignes vides (limités de paragraphe) avant le découpage par longueur.
- `channels.bluebubbles.mediaMaxMb` : limité de médias entrants en Mo (par défaut : 8).
- `channels.bluebubbles.mediaLocalRoots` : liste autorisée explicite des répertoires locaux absolus autorisés pour les chemins de médias locaux sortants. Les envois par chemin local sont refuses par défaut sauf si ceci est configuré. Remplacement par compte : `channels.bluebubbles.accounts.<accountId>.mediaLocalRoots`.
- `channels.bluebubbles.historyLimit` : nombre maximum de messages de groupe pour le contexte (0 désactive).
- `channels.bluebubbles.dmHistoryLimit` : limité d'historique des messages privés.
- `channels.bluebubbles.actions` : activer/désactiver des actions spécifiques.
- `channels.bluebubbles.accounts` : configuration multi-comptes.

Options globales associées :

- `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`).
- `messages.responsePrefix`.

## Adressage / cibles de livraison

Préférez `chat_guid` pour un routage stable :

- `chat_guid:iMessage;-;+15555550123` (préféré pour les groupes)
- `chat_id:123`
- `chat_identifier:...`
- Identifiants directs : `+15555550123`, `user@example.com`
  - Si un identifiant direct n'a pas de discussion de message privé existante, OpenClaw en creera une via `POST /api/v1/chat/new`. Cela nécessite que l'API privée BlueBubbles soit activée.

## Sécurité

- Les requêtes webhook sont authentifiees en comparant les paramètres de requête/en-tetes `guid`/`password` avec `channels.bluebubbles.