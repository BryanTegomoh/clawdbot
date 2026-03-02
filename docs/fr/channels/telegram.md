---
summary: "Statut de prise en charge du bot Telegram, capacités et configuration"
read_when:
  - Travail sur les fonctionnalités Telegram ou les webhooks
title: "Telegram"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/telegram.md
  workflow: manual
---

# Telegram (Bot API)

Statut : prêt pour la production pour les messages privés et les groupes de bots via grammY. Le long polling est le mode par défaut ; le mode webhook est optionnel.

<CardGroup cols={3}>
  <Card title="Appairage" icon="link" href="/channels/pairing">
    La politique de messages privés par défaut pour Telegram est l'appairage.
  </Card>
  <Card title="Dépannage des canaux" icon="wrench" href="/channels/troubleshooting">
    Diagnostics et guides de réparation inter-canaux.
  </Card>
  <Card title="Configuration du Gateway" icon="settings" href="/gateway/configuration">
    Modèles et exemples complets de configuration de canal.
  </Card>
</CardGroup>

## Configuration rapide

<Steps>
  <Step title="Créer le jeton du bot dans BotFather">
    Ouvrez Telegram et discutez avec **@BotFather** (confirmez que l'identifiant est exactement `@BotFather`).

    Exécutez `/newbot`, suivez les instructions et sauvegardez le jeton.

  </Step>

  <Step title="Configurer le jeton et la politique de messages privés">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    Repli par variable d'environnement : `TELEGRAM_BOT_TOKEN=...` (compte par défaut uniquement).
    Telegram n'utilisé **pas** `openclaw channels login telegram` ; configurez le jeton dans la config/env, puis démarrez le gateway.

  </Step>

  <Step title="Démarrer le gateway et approuver le premier message privé">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    Les codes d'appairage expirent après 1 heure.

  </Step>

  <Step title="Ajouter le bot a un groupe">
    Ajoutez le bot a votre groupe, puis définissez `channels.telegram.groups` et `groupPolicy` pour correspondre a votre modèle d'accès.
  </Step>
</Steps>

<Note>
L'ordre de résolution des jetons est sensible au compte. En pratique, les valeurs de configuration prennent le pas sur le repli par variable d'environnement, et `TELEGRAM_BOT_TOKEN` s'applique uniquement au compte par défaut.
</Note>

## Paramètres côté Telegram

<AccordionGroup>
  <Accordion title="Mode de confidentialité et visibilité de groupe">
    Les bots Telegram utilisent par défaut le **mode de confidentialité**, qui limite les messages de groupe qu'ils reçoivent.

    Si le bot doit voir tous les messages du groupe, vous pouvez :

    - désactiver le mode de confidentialité via `/setprivacy`, ou
    - rendre le bot administrateur du groupe.

    Lorsque vous basculez le mode de confidentialité, supprimez puis re-ajoutez le bot dans chaque groupe pour que Telegram applique le changement.

  </Accordion>

  <Accordion title="Permissions de groupe">
    Le statut d'administrateur est contrôle dans les paramètres du groupe Telegram.

    Les bots administrateurs reçoivent tous les messages du groupe, ce qui est utile pour un comportement de groupe permanent.

  </Accordion>

  <Accordion title="Bascules utiles de BotFather">

    - `/setjoingroups` pour autoriser/refuser les ajouts aux groupes
    - `/setprivacy` pour le comportement de visibilité de groupe

  </Accordion>
</AccordionGroup>

## Contrôle d'accès et activation

<Tabs>
  <Tab title="Politique de messages privés">
    `channels.telegram.dmPolicy` contrôle l'accès aux messages directs :

    - `pairing` (par défaut)
    - `allowlist`
    - `open` (nécessite que `allowFrom` inclue `"*"`)
    - `disabled`

    `channels.telegram.allowFrom` accepte les identifiants utilisateur Telegram numériques. Les préfixes `telegram:` / `tg:` sont acceptés et normalisés.
    L'assistant d'intégration accepte la saisie `@username` et la resout en identifiants numériques.
    Si vous avez mis a jour et que votre configuration contient des entrées de liste autorisée `@username`, exécutez `openclaw doctor --fix` pour les résoudre (meilleur effort ; nécessite un jeton de bot Telegram).

    ### Trouver votre identifiant utilisateur Telegram

    Plus sur (sans bot tiers) :

    1. Envoyez un message privé a votre bot.
    2. Exécutez `openclaw logs --follow`.
    3. Lisez `from.id`.

    Méthode officielle Bot API :

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    Méthode tierce (moins privée) : `@userinfobot` ou `@getidsbot`.

  </Tab>

  <Tab title="Politique de groupe et listes autorisées">
    Il y a deux contrôles indépendants :

    1. **Quels groupes sont autorisés** (`channels.telegram.groups`)
       - pas de configuration `groups` : tous les groupes autorisés
       - `groups` configuré : agit comme liste autorisée (identifiants explicites ou `"*"`)

    2. **Quels expéditeurs sont autorisés dans les groupes** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (par défaut)
       - `disabled`

    `groupAllowFrom` est utilisé pour le filtrage des expéditeurs de groupe. S'il n'est pas défini, Telegram se rabat sur `allowFrom`.
    Les entrées `groupAllowFrom` doivent être des identifiants utilisateur Telegram numériques.
    Note d'exécution : si `channels.telegram` est complètement absent, l'exécution se rabat sur `groupPolicy="allowlist"` pour l'évaluation de la politique de groupe (même si `channels.defaults.groupPolicy` est défini).

    Exemple : autoriser tout membre dans un groupe spécifique :

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  </Tab>

  <Tab title="Comportement de mention">
    Les réponses de groupe nécessitent une mention par défaut.

    La mention peut provenir de :

    - la mention native `@botusername`, ou
    - les modèles de mention dans :
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`

    Bascules de commande au niveau de la session :

    - `/activation always`
    - `/activation mention`

    Celles-ci mettent a jour uniquement l'état de la session. Utilisez la configuration pour la persistance.

    Exemple de configuration persistante :

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    Obtenir l'identifiant du groupe :

    - transférez un message du groupe a `@userinfobot` / `@getidsbot`
    - ou lisez `chat.id` depuis `openclaw logs --follow`
    - ou inspectez Bot API `getUpdates`

  </Tab>
</Tabs>

## Comportement a l'exécution

- Telegram est géré par le processus du gateway.
- Le routage est déterministe : les messages entrants Telegram répondent vers Telegram (le modèle ne choisit pas les canaux).
- Les messages entrants se normalisent dans l'enveloppe de canal partagee avec les métadonnées de réponse et les espaces réservés pour les médias.
- Les sessions de groupe sont isolees par identifiant de groupe. Les sujets de forum ajoutent `:topic:<threadId>` pour garder les sujets isoles.
- Les messages privés peuvent contenir `message_thread_id` ; OpenClaw les achemine avec des clés de session sensibles aux fils et preserve l'identifiant de fil pour les réponses.
- Le long polling utilise le runner grammY avec un sequencement par chat/fil. La concurrence globale du sink du runner utilisé `agents.defaults.maxConcurrent`.
- Telegram Bot API n'a pas de support d'accuse de reception (`sendReadReceipts` ne s'applique pas).

## Référence des fonctionnalités

<AccordionGroup>
  <Accordion title="Aperçu en direct du flux (modifications de messages)">
    OpenClaw peut diffuser des réponses partielles en envoyant un message Telegram temporaire et en le modifiant au fur et a mesure que le texte arrive.

    Prerequis :

    - `channels.telegram.streaming` est `off | partial | block | progress` (par défaut : `off`)
    - `progress` correspond a `partial` sur Telegram (compatibilité avec le nommage inter-canal)
    - l'ancien `channels.telegram.streamMode` et les valeurs booleennes `streaming` sont auto-mappees

    Cela fonctionne dans les discussions directes et les groupes/sujets.

    Pour les réponses textuelles uniquement, OpenClaw garde le même message d'aperçu et effectué une modification finale sur place (pas de second message).

    Pour les réponses complexes (par exemple les payloads médias), OpenClaw se rabat sur la livraison finale normale puis nettoie le message d'aperçu.

    La diffusion d'aperçu est séparée de la diffusion par blocs. Lorsque la diffusion par blocs est explicitement activée pour Telegram, OpenClaw saute le flux d'aperçu pour eviter la double diffusion.

    Flux de raisonnement Telegram uniquement :

    - `/reasoning stream` envoie le raisonnement a l'aperçu en direct pendant la génération
    - la réponse finale est envoyée sans le texte de raisonnement

  </Accordion>

  <Accordion title="Formatage et repli HTML">
    Le texte sortant utilisé le `parse_mode: "HTML"` de Telegram.

    - Le texte de type Markdown est rendu en HTML compatible Telegram.
    - Le HTML brut du modèle est echappe pour réduire les échecs d'analyse Telegram.
    - Si Telegram rejette le HTML analyse, OpenClaw réessaie en texte brut.

    Les aperçus de liens sont activés par défaut et peuvent être désactivés avec `channels.telegram.linkPreview: false`.

  </Accordion>

  <Accordion title="Commandes natives et commandes personnalisées">
    L'enregistrement du menu de commandes Telegram est géré au démarrage avec `setMyCommands`.

    Valeurs par défaut des commandes natives :

    - `commands.native: "auto"` activé les commandes natives pour Telegram

    Ajouter des entrées de menu de commandes personnalisées :

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    Règles :

    - les noms sont normalisés (suppression du `/` initial, mise en minuscules)
    - modèle valide : `a-z`, `0-9`, `_`, longueur `1..32`
    - les commandes personnalisées ne peuvent pas remplacer les commandes natives
    - les conflits/doublons sont ignorés et journalises

    Notes :

    - les commandes personnalisées sont des entrées de menu uniquement ; elles n'implementent pas automatiquement de comportement
    - les commandes de plugin/competence peuvent toujours fonctionner lorsqu'elles sont tapees même si elles ne sont pas affichees dans le menu Telegram

    Si les commandes natives sont désactivées, les commandes intégrées sont supprimées. Les commandes personnalisées/plugin peuvent toujours s'enregistrer si elles sont configurées.

    Échec de configuration courant :

    - `setMyCommands failed` signifie généralement que le DNS/HTTPS sortant vers `api.telegram.org` est bloque.

    ### Commandes d'appairage d'appareil (plugin `device-pair`)

    Lorsque le plugin `device-pair` est installe :

    1. `/pair` généré le code de configuration
    2. collez le code dans l'application iOS
    3. `/pair approve` approuve la dernière demande en attente

    Plus de details : [Appairage](/channels/pairing#pair-via-telegram-recommended-for-ios).

  </Accordion>

  <Accordion title="Boutons en ligne">
    Configurer le périmètre du clavier en ligne :

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    Remplacement par compte :

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    Périmètres :

    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (par défaut)

    L'ancien `capabilities: ["inlineButtons"]` correspond a `inlineButtons: "all"`.

    Exemple d'action de message :

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    Les clics de callback sont passes a l'agent sous forme de texte :
    `callback_data: <value>`

  </Accordion>

  <Accordion title="Actions de message Telegram pour les agents et l'automatisation">
    Les actions d'outil Telegram incluent :

    - `sendMessage` (`to`, `content`, optionnel `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)

    Les actions de message du canal exposent des alias ergonomiques (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).

    Contrôles de filtrage :

    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (par défaut : désactivé)

    Semantique de suppression de reaction : [/tools/reactions](/tools/reactions)

  </Accordion>

  <Accordion title="Balises de fil de réponse">
    Telegram prend en charge les balises explicites de fil de réponse dans la sortie générée :

    - `[[reply_to_current]]` répond au message declencheur
    - `[[reply_to:<id>]]` répond a un identifiant de message Telegram spécifique

    `channels.telegram.replyToMode` contrôle le traitement :

    - `off` (par défaut)
    - `first`
    - `all`

    Note : `off` désactive le fil de réponse implicite. Les balises explicites `[[reply_to_*]]` sont toujours honorees.

  </Accordion>

  <Accordion title="Sujets de forum et comportement des fils">
    Supergroupes de forum :

    - les clés de session de sujet ajoutent `:topic:<threadId>`
    - les réponses et la saisie ciblent le fil du sujet
    - chemin de configuration du sujet :
      `channels.telegram.groups.<chatId>.topics.<threadId>`

    Cas special du sujet général (`threadId=1`) :

    - les envois de messages omettent `message_thread_id` (Telegram rejette `sendMessage(...thread_id=1)`)
    - les actions de saisie incluent toujours `message_thread_id`

    Héritage des sujets : les entrées de sujet héritent des paramètres du groupe sauf si elles sont remplacees (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).

    Le contexte de modèle inclut :

    - `MessageThreadId`
    - `IsForum`

    Comportement des fils en messages privés :

    - les discussions privées avec `message_thread_id` gardent le routage de messages privés mais utilisent des clés de session sensibles aux fils et des cibles de réponse preservant l'identifiant de fil.

  </Accordion>

  <Accordion title="Audio, video et stickers">
    ### Messages audio

    Telegram distingue les notes vocales des fichiers audio.

    - par défaut : comportement de fichier audio
    - balise `[[audio_as_voice]]` dans la réponse de l'agent pour forcer l'envoi en note vocale

    Exemple d'action de message :

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ### Messages video

    Telegram distingue les fichiers video des notes video.

    Exemple d'action de message :

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    Les notes video ne prennent pas en charge les légendes ; le texte de message fourni est envoyé séparément.

    ### Stickers

    Traitement des stickers entrants :

    - WEBP statique : téléchargé et traite (espace réservé `<média:sticker>`)
    - TGS anime : ignoré
    - WEBM video : ignoré

    Champs de contexte des stickers :

    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`

    Fichier de cache des stickers :

    - `~/.openclaw/telegram/sticker-cache.json`

    Les stickers sont decrits une fois (lorsque c'est possible) et mis en cache pour réduire les appels de vision repetes.

    Activer les actions de stickers :

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    Action d'envoi de sticker :

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    Rechercher des stickers en cache :

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  </Accordion>

  <Accordion title="Notifications de reaction">
    Les reactions Telegram arrivent en tant que mises a jour `message_reaction` (séparées des payloads de message).

    Lorsqu'elles sont activées, OpenClaw met en file d'attente des événements système comme :

    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`

    Configuration :

    - `channels.telegram.reactionNotifications` : `off | own | all` (par défaut : `own`)
    - `channels.telegram.reactionLevel` : `off | ack | minimal | extensive` (par défaut : `minimal`)

    Notes :

    - `own` signifie uniquement les reactions des utilisateurs aux messages envoyés par le bot (meilleur effort via le cache des messages envoyés).
    - Telegram ne fournit pas d'identifiants de fil dans les mises a jour de reaction.
      - les groupes non-forum sont achemines vers la session de discussion du groupe
      - les groupes de forum sont achemines vers la session du sujet général du groupe (`:topic:1`), pas vers le sujet d'origine exact

    `allowed_updates` pour le polling/webhook inclut `message_reaction` automatiquement.

  </Accordion>

  <Accordion title="Réactions d'accusé de reception">
    `ackReaction` envoie un emoji d'accusé de reception pendant qu'OpenClaw traite un message entrant.

    Ordre de résolution :

    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - repli sur l'emoji d'identité de l'agent (`agents.list[].identity.emoji`, sinon "👀")

    Notes :

    - Telegram attend un emoji unicode (par exemple "👀").
    - Utilisez `""` pour désactiver la reaction pour un canal ou un compte.

  </Accordion>

  <Accordion title="Écritures de configuration depuis les événements et commandes Telegram">
    Les écritures de configuration du canal sont activées par défaut (`configWrites !== false`).

    Les écritures declenchees par Telegram incluent :

    - les événements de migration de groupe (`migrate_to_chat_id`) pour mettre a jour `channels.telegram.groups`
    - `/config set` et `/config unset` (nécessite l'activation de la commande)

    Désactiver :

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  </Accordion>

  <Accordion title="Long polling vs webhook">
    Par défaut : long polling.

    Mode webhook :

    - définir `channels.telegram.webhookUrl`
    - définir `channels.telegram.webhookSecret` (requis lorsque l'URL webhook est définie)
    - optionnel `channels.telegram.webhookPath` (par défaut `/telegram-webhook`)
    - optionnel `channels.telegram.webhookHost` (par défaut `127.0.0.1`)

    L'écouteur local par défaut pour le mode webhook se lie a `127.0.0.1:8787`.

    Si votre point de terminaison public differe, placez un proxy inverse devant et pointez `webhookUrl` vers l'URL publique.
    Définissez `webhookHost` (par exemple `0.0.0.0`) lorsque vous avez intentionnellement besoin d'un accès externe.

  </Accordion>

  <Accordion title="Limités, nouvelle tentative et cibles CLI">
    - `channels.telegram.textChunkLimit` par défaut est 4000.
    - `channels.telegram.chunkMode="newline"` préféré les limités de paragraphe (lignes vides) avant le découpage par longueur.
    - `channels.telegram.mediaMaxMb` (par défaut 5) limité la taille de téléchargement/traitement des médias entrants Telegram.
    - `channels.telegram.timeoutSeconds` remplace le timeout du client API Telegram (si non défini, la valeur par défaut de grammY s'applique).
    - l'historique de contexte du groupe utilisé `channels.telegram.historyLimit` ou `messages.groupChat.historyLimit` (par défaut 50) ; `0` désactive.
    - Contrôles de l'historique des messages privés :
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - les nouvelles tentatives de l'API Telegram sortante sont configurables via `channels.telegram.retry`.

    La cible d'envoi CLI peut être un identifiant de chat numérique ou un nom d'utilisateur :

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  </Accordion>
</AccordionGroup>

## Dépannage

<AccordionGroup>
  <Accordion title="Le bot ne répond pas aux messages de groupe sans mention">

    - Si `requireMention=false`, le mode de confidentialité Telegram doit permettre une visibilité complète.
      - BotFather : `/setprivacy` -> Disable
      - puis supprimez + re-ajoutez le bot au groupe
    - `openclaw channels status` avertit lorsque la configuration attend des messages de groupe sans mention.
    - `openclaw channels status --probe` peut vérifier les identifiants de groupe numériques explicites ; le joker `"*"` ne peut pas être sonde pour l'appartenance.
    - test rapide de session : `/activation always`.

  </Accordion>

  <Accordion title="Le bot ne voit pas du tout les messages de groupe">

    - quand `channels.telegram.groups` existe, le groupe doit être liste (ou inclure `"*"`)
    - vérifier l'appartenance du bot au groupe
    - examiner les logs : `openclaw logs --follow` pour les raisons d'exclusion

  </Accordion>

  <Accordion title="Les commandes fonctionnent partiellement ou pas du tout">

    - autorisez votre identité d'expéditeur (appairage et/ou `allowFrom` numérique)
    - l'autorisation de commande s'applique toujours même lorsque la politique de groupe est `open`
    - `setMyCommands failed` indique généralement des problèmes d'accessibilité DNS/HTTPS vers `api.telegram.org`

  </Accordion>

  <Accordion title="Instabilite du polling ou du réseau">

    - Node 22+ + fetch personnalisé/proxy peut déclencher un comportement d'abandon immédiat si les types AbortSignal ne correspondent pas.
    - Certains hebergeurs resolvent `api.telegram.org` en IPv6 en premier ; un egress IPv6 defaillant peut causer des échecs intermittents de l'API Telegram.
    - Si les logs contiennent `TypeError: fetch failed` ou `Network request for 'getUpdates' failed!`, OpenClaw réessaie maintenant ces erreurs comme des erreurs réseau recuperables.
    - Sur les hebergeurs VPS avec un egress/TLS direct instable, acheminez les appels API Telegram via `channels.telegram.proxy` :

```yaml
channels:
  telegram:
    proxy: socks5://user:pass@proxy-host:1080
```

    - Node 22+ utilisé par défaut `autoSelectFamily=true` (sauf WSL2) et `dnsResultOrder=ipv4first`.
    - Si votre hebergeur est WSL2 ou fonctionne explicitement mieux avec un comportement IPv4 uniquement, forcez la sélection de famille :

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

    - Variables d'environnement de remplacement (temporaire) :
      - `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
      - `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
      - `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
    - Valider les réponses DNS :

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  </Accordion>
</AccordionGroup>

Plus d'aide : [Dépannage des canaux](/channels/troubleshooting).

## Pointeurs de référence de configuration Telegram

Référence principale :

- `channels.telegram.enabled` : activer/désactiver le démarrage du canal.
- `channels.telegram.botToken` : jeton du bot (BotFather).
- `channels.telegram.tokenFile` : lire le jeton depuis un chemin de fichier.
- `channels.telegram.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
- `channels.telegram.allowFrom` : liste autorisée de messages privés (identifiants utilisateur Telegram numériques). `open` nécessite `"*"`. `openclaw doctor --fix` peut résoudre les anciennes entrées `@username` en identifiants.
- `channels.telegram.groupPolicy` : `open | allowlist | disabled` (par défaut : allowlist).
- `channels.telegram.groupAllowFrom` : liste autorisée d'expéditeurs de groupe (identifiants utilisateur Telegram numériques). `openclaw doctor --fix` peut résoudre les anciennes entrées `@username` en identifiants.
- Precedence multi-comptes :
  - `channels.telegram.accounts.default.allowFrom` et `channels.telegram.accounts.default.groupAllowFrom` s'appliquent uniquement au compte `default`.
  - Les comptes nommes héritent de `channels.telegram.allowFrom` et `channels.telegram.groupAllowFrom` lorsque les valeurs au niveau du compte ne sont pas définies.
  - Les comptes nommes n'héritent pas de `channels.telegram.accounts.default.allowFrom` / `groupAllowFrom`.
- `channels.telegram.groups` : valeurs par défaut par groupe + liste autorisée (utilisez `"*"` pour les valeurs par défaut globales).
  - `channels.telegram.groups.<id>.groupPolicy` : remplacement par groupe de groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention` : valeur par défaut du filtrage par mention.
  - `channels.telegram.groups.<id>.skills` : filtré de competences (omettre = toutes les competences, vide = aucune).
  - `channels.telegram.groups.<id>.allowFrom` : remplacement de la liste autorisée d'expéditeurs par groupe.
  - `channels.telegram.groups.<id>.systemPrompt` : prompt système supplementaire pour le groupe.
  - `channels.telegram.groups.<id>.enabled` : désactiver le groupe lorsque `false`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*` : remplacements par sujet (mêmes champs que le groupe).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy` : remplacement par sujet de groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention` : remplacement du filtrage par mention par sujet.
- `channels.telegram.capabilities.inlineButtons` : `off | dm | group | all | allowlist` (par défaut : allowlist).
- `channels.telegram.accounts.<account>.capabilities.inlineButtons` : remplacement par compte.
- `channels.telegram.replyToMode` : `off | first | all` (par défaut : `off`).
- `channels.telegram.textChunkLimit` : taille de découpage sortant (caractères).
- `channels.telegram.chunkMode` : `length` (par défaut) ou `newline` pour decouper sur les lignes vides (limités de paragraphe) avant le découpage par longueur.
- `channels.telegram.linkPreview` : basculer les aperçus de liens pour les messages sortants (par défaut : true).
- `channels.telegram.streaming` : `off | partial | block | progress` (aperçu en direct ; par défaut : `off` ; `progress` correspond a `partial`).
- `channels.telegram.mediaMaxMb` : limité de médias entrants/sortants (Mo).
- `channels.telegram.retry` : politique de nouvelle tentative pour les appels API Telegram sortants (attempts, minDelayMs, maxDelayMs, jitter).
- `channels.telegram.network.autoSelectFamily` : remplacer Node autoSelectFamily (true=activer, false=désactiver). Par défaut activé sur Node 22+, avec WSL2 par défaut désactivé.
- `channels.telegram.network.dnsResultOrder` : remplacer l'ordre des résultats DNS (`ipv4first` ou `verbatim`). Par défaut `ipv4first` sur Node 22+.
- `channels.telegram.proxy` : URL du proxy pour les appels Bot API (SOCKS/HTTP).
- `channels.telegram.webhookUrl` : activer le mode webhook (nécessite `channels.telegram.webhookSecret`).
- `channels.telegram.webhookSecret` : secret du webhook (requis lorsque webhookUrl est défini).
- `channels.telegram.webhookPath` : chemin local du webhook (par défaut `/telegram-webhook`).
- `channels.telegram.webhookHost` : hôte de liaison du webhook local (par défaut `127.0.0.1`).
- `channels.telegram.actions.reactions` : contrôle des reactions d'outil Telegram.
- `channels.telegram.actions.sendMessage` : contrôle des envois de messages d'outil Telegram.
- `channels.telegram.actions.deleteMessage` : contrôle des suppressions de messages d'outil Telegram.
- `channels.telegram.actions.sticker` : contrôle des actions de stickers Telegram (envoi et recherche) (par défaut : false).
- `channels.telegram.reactionNotifications` : `off | own | all` : contrôle des reactions qui declenchent des événements système (par défaut : `own` si non défini).
- `channels.telegram.reactionLevel` : `off | ack | minimal | extensive` : contrôle de la capacité de reaction de l'agent (par défaut : `minimal` si non défini).

- [Référence de configuration - Telegram](/gateway/configuration-reference#telegram)

Champs Telegram a fort signal :

- démarrage/auth : `enabled`, `botToken`, `tokenFile`, `accounts.*`
- contrôle d'accès : `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- commande/menu : `commands.native`, `customCommands`
- fils/réponses : `replyToMode`
- streaming : `streaming` (aperçu), `blockStreaming`
- formatage/livraison : `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- média/réseau : `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook : `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- actions/capacités : `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- reactions : `reactionNotifications`, `reactionLevel`
- écritures/historique : `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Liens associés

- [Appairage](/channels/pairing)
- [Routage des canaux](/channels/channel-routing)
- [Routage multi-agent](/concepts/multi-agent)
- [Dépannage](/channels/troubleshooting)
