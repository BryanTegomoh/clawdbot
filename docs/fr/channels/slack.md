---
summary: "Configuration et comportement de Slack (Socket Mode + HTTP Events API)"
read_when:
  - Configuration de Slack ou débogage du mode socket/HTTP Slack
title: "Slack"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/slack.md
  workflow: manual
---

# Slack

Statut : prêt pour la production pour les Messages privés + canaux via les intégrations d'applications Slack. Le mode par défaut est Socket Mode ; le mode HTTP Events API est également pris en charge.

<CardGroup cols={3}>
  <Card title="Appairage" icon="link" href="/channels/pairing">
    Les Messages privés Slack utilisent le mode appairage par défaut.
  </Card>
  <Card title="Commandes slash" icon="terminal" href="/tools/slash-commands">
    Comportement des commandes natives et catalogue de commandes.
  </Card>
  <Card title="Dépannage des canaux" icon="wrench" href="/channels/troubleshooting">
    Diagnostics multi-canaux et guides de réparation.
  </Card>
</CardGroup>

## Configuration rapide

<Tabs>
  <Tab title="Socket Mode (par défaut)">
    <Steps>
      <Step title="Créer l'application Slack et les jetons">
        Dans les paramètres de l'application Slack :

        - activez le **Socket Mode**
        - créez un **App Token** (`xapp-...`) avec `connections:write`
        - installez l'application et copiez le **Bot Token** (`xoxb-...`)
      </Step>

      <Step title="Configurer OpenClaw">

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        Repli sur les variables d'environnement (compte par défaut uniquement) :

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

      </Step>

      <Step title="S'abonner aux événements de l'application">
        Abonnez les événements du bot pour :

        - `app_mention`
        - `message.channels`, `message.groups`, `message.im`, `message.mpim`
        - `reaction_added`, `reaction_removed`
        - `member_joined_channel`, `member_left_channel`
        - `channel_rename`
        - `pin_added`, `pin_removed`

        Activez également l'onglet **Messages** dans App Home pour les Messages privés.
      </Step>

      <Step title="Démarrer le Gateway">

```bash
openclaw gateway
```

      </Step>
    </Steps>

  </Tab>

  <Tab title="Mode HTTP Events API">
    <Steps>
      <Step title="Configurer l'application Slack pour HTTP">

        - définissez le mode sur HTTP (`channels.slack.mode="http"`)
        - copiez le **Signing Secret** de Slack
        - définissez l'URL de requête des Event Subscriptions + Interactivity + Slash command sur le même chemin webhook (par défaut `/slack/events`)

      </Step>

      <Step title="Configurer OpenClaw en mode HTTP">

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      </Step>

      <Step title="Utiliser des chemins webhook uniques pour le multi-compte HTTP">
        Le mode HTTP par compte est pris en charge.

        Donnez à chaque compte un `webhookPath` distinct pour éviter les collisions d'enregistrement.
      </Step>
    </Steps>

  </Tab>
</Tabs>

## Modèle de jetons

- `botToken` + `appToken` sont requis pour le Socket Mode.
- Le mode HTTP nécessite `botToken` + `signingSecret`.
- Les jetons de configuration écrasent le repli sur les variables d'environnement.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` en repli d'environnement s'appliquent uniquement au compte par défaut.
- `userToken` (`xoxp-...`) est uniquement en configuration (pas de repli d'environnement) et est en lecture seule par défaut (`userTokenReadOnly: true`).
- Optionnel : ajoutez `chat:write.customize` si vous voulez que les messages sortants utilisent l'identité de l'agent actif (`username` et icône personnalisés). `icon_emoji` utilisé la syntaxe `:emoji_name:`.

<Tip>
Pour les actions/lectures de répertoire, le jeton utilisateur peut être préféré lorsqu'il est configuré. Pour les écritures, le jeton bot reste préféré ; les écritures par jeton utilisateur ne sont autorisées que lorsque `userTokenReadOnly: false` et que le jeton bot n'est pas disponible.
</Tip>

## Contrôle d'accès et routage

<Tabs>
  <Tab title="Politique de MP">
    `channels.slack.dmPolicy` contrôle l'accès aux Messages privés (ancien : `channels.slack.dm.policy`) :

    - `pairing` (par défaut)
    - `allowlist`
    - `open` (nécessite que `channels.slack.allowFrom` inclue `"*"` ; ancien : `channels.slack.dm.allowFrom`)
    - `disabled`

    Options des Messages privés :

    - `dm.enabled` (par défaut true)
    - `channels.slack.allowFrom` (préféré)
    - `dm.allowFrom` (ancien)
    - `dm.groupEnabled` (MP de groupe par défaut false)
    - `dm.groupChannels` (liste autorisée MPIM optionnelle)

    Precedence multi-comptes :

    - `channels.slack.accounts.default.allowFrom` s'applique uniquement au compte `default`.
    - Les comptes nommes héritent de `channels.slack.allowFrom` lorsque leur propre `allowFrom` n'est pas défini.
    - Les comptes nommes n'héritent pas de `channels.slack.accounts.default.allowFrom`.

    L'appairage dans les MP utilisé `openclaw pairing approve slack <code>`.

  </Tab>

  <Tab title="Politique de canal">
    `channels.slack.groupPolicy` contrôle la gestion des canaux :

    - `open`
    - `allowlist`
    - `disabled`

    La liste autorisée des canaux se trouve sous `channels.slack.channels`.

    Note d'exécution : si `channels.slack` est complètement absent (configuration par variables d'environnement uniquement), l'exécution revient à `groupPolicy="allowlist"` et enregistre un avertissement (même si `channels.defaults.groupPolicy` est défini).

    Résolution nom/ID :

    - les entrées de la liste autorisée de canaux et de MP sont résolues au démarrage lorsque l'accès par jeton le permet
    - les entrées non résolues sont conservées telles que configurées
    - la correspondance d'autorisation entrante est par ID en premier par défaut ; la correspondance directe par nom d'utilisateur/slug nécessite `channels.slack.dangerouslyAllowNameMatching: true`

  </Tab>

  <Tab title="Mentions et utilisateurs du canal">
    Les messages de canal sont filtrés par mention par défaut.

    Sources de mention :

    - mention explicite de l'application (`<@botId>`)
    - motifs regex de mention (`agents.list[].groupChat.mentionPatterns`, repli `messages.groupChat.mentionPatterns`)
    - comportement implicite de réponse au bot dans un fil

    Contrôles par canal (`channels.slack.channels.<id|name>`) :

    - `requireMention`
    - `users` (liste autorisée)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    - Format de clé `toolsBySender` : `id:`, `e164:`, `username:`, `name:`, ou `"*"` générique
      (les clés héritées sans préfixe correspondent toujours uniquement à `id:`)

  </Tab>
</Tabs>

## Commandes et comportement slash

- Le mode auto des commandes natives est **désactivé** pour Slack (`commands.native: "auto"` n'activé pas les commandes natives Slack).
- Activez les gestionnaires de commandes natives Slack avec `channels.slack.commands.native: true` (ou globalement `commands.native: true`).
- Lorsque les commandes natives sont activées, enregistrez les commandes slash correspondantes dans Slack (noms `/<commande>`).
- Si les commandes natives ne sont pas activées, vous pouvez exécuter une seule commande slash configurée via `channels.slack.slashCommand`.
- Les menus d'arguments des commandes natives adaptent maintenant leur stratégie de rendu :
  - jusqu'à 5 options : blocs de boutons
  - 6-100 options : menu de sélection statique
  - plus de 100 options : sélection externe avec filtrage asynchrone des options lorsque les gestionnaires d'options d'interactivité sont disponibles
  - si les valeurs d'option encodées dépassent les limités Slack, le flux revient aux boutons
- Pour les longs payloads d'options, les menus d'arguments de commande slash utilisent un dialogue de confirmation avant d'envoyer la valeur sélectionnée.

Paramètres par défaut des commandes slash :

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Les sessions slash utilisent des clés isolées :

- `agent:<agentId>:slack:slash:<userId>`

et routent toujours l'exécution de la commande vers la session de conversation cible (`CommandTargetSessionKey`).

## Fils de discussion, sessions et tags de réponse

- Les MP sont routés comme `direct` ; les canaux comme `channel` ; les MPIM comme `group`.
- Avec le `session.dmScope=main` par défaut, les MP Slack se regroupent dans la session principale de l'agent.
- Sessions de canal : `agent:<agentId>:slack:channel:<channelId>`.
- Les réponses dans les fils peuvent créer des suffixes de session de fil (`:thread:<threadTs>`) le cas échéant.
- `channels.slack.thread.historyScope` par défaut est `thread` ; `thread.inheritParent` par défaut est `false`.
- `channels.slack.thread.initialHistoryLimit` contrôle combien de messages existants du fil sont récupérés au démarrage d'une nouvelle session de fil (par défaut `20` ; définissez `0` pour désactiver).

Contrôles de réponse dans les fils :

- `channels.slack.replyToMode` : `off|first|all` (par défaut `off`)
- `channels.slack.replyToModeByChatType` : par `direct|group|channel`
- repli hérité pour les conversations directes : `channels.slack.dm.replyToMode`

Les tags de réponse manuels sont pris en charge :

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Note : `replyToMode="off"` désactive **toutes** les réponses dans les fils de Slack, y compris les tags explicites `[[reply_to_*]]`. Cela diffère de Telegram, où les tags explicites sont toujours honorés en mode `"off"`. La différence reflète les modèles de fils des plateformes : les fils Slack masquent les messages du canal, tandis que les réponses Telegram restent visibles dans le flux principal du chat.

## Médias, découpage et livraison

<AccordionGroup>
  <Accordion title="Pièces jointes entrantes">
    Les pièces jointes Slack sont téléchargées depuis les URL privées hébergées par Slack (flux de requêtes authentifiées par jeton) et écrites dans le stockage média lorsque le téléchargement réussit et que les limités de taille le permettent.

    La taille maximale entrante par défaut est `20MB` sauf si elle est remplacée par `channels.slack.mediaMaxMb`.

  </Accordion>

  <Accordion title="Texte et fichiers sortants">
    - les morceaux de texte utilisent `channels.slack.textChunkLimit` (par défaut 4000)
    - `channels.slack.chunkMode="newline"` activé le découpage par paragraphe en priorité
    - les envois de fichiers utilisent les API d'upload Slack et peuvent inclure des réponses en fil (`thread_ts`)
    - la taille maximale des médias sortants suit `channels.slack.mediaMaxMb` lorsqu'il est configuré ; sinon les envois de canal utilisent les valeurs par défaut par type MIME du pipeline média
  </Accordion>

  <Accordion title="Cibles de livraison">
    Cibles explicites préférées :

    - `user:<id>` pour les Messages privés
    - `channel:<id>` pour les canaux

    Les MP Slack sont ouverts via les API de conversation Slack lors de l'envoi vers des cibles utilisateur.

  </Accordion>
</AccordionGroup>

## Actions et portes

Les actions Slack sont contrôlées par `channels.slack.actions.*`.

Groupes d'actions disponibles dans l'outillage Slack actuel :

| Groupe     | Par défaut |
| ---------- | ---------- |
| messages   | activé     |
| reactions  | activé     |
| pins       | activé     |
| memberInfo | activé     |
| emojiList  | activé     |

## Événements et comportement opérationnel

- Les modifications/suppressions/diffusions de messages dans les fils sont mappés en événements système.
- Les événements d'ajout/suppression de réaction sont mappés en événements système.
- Les événements d'arrivée/départ de membre, création/renommage de canal, et ajout/suppression d'épingle sont mappés en événements système.
- Les mises à jour de statut de fil assistant (pour les indicateurs "est en train d'écrire..." dans les fils) utilisent `assistant.threads.setStatus` et nécessitent le scope bot `assistant:write`.
- `channel_id_changed` peut migrer les clés de configuration de canal lorsque `configWrites` est activé.
- Les métadonnées de sujet/objectif de canal sont traitées comme contexte non fiable et peuvent être injectées dans le contexte de routage.
- Les actions de bloc et les interactions modales émettent des événements système structurés `Slack interaction: ...` avec des champs de payload enrichis :
  - actions de bloc : valeurs sélectionnées, labels, valeurs de sélecteur, et métadonnées `workflow_*`
  - événements modaux `view_submission` et `view_closed` avec métadonnées de canal routé et entrées de formulaire

## Réactions d'accusé de réception

`ackReaction` envoie un emoji d'accusé de réception pendant qu'OpenClaw traite un message entrant.

Ordre de résolution :

- `channels.slack.accounts.<accountId>.ackReaction`
- `channels.slack.ackReaction`
- `messages.ackReaction`
- repli sur l'emoji d'identité de l'agent (`agents.list[].identity.emoji`, sinon "👀")

Notes :

- Slack attend des shortcodes (par exemple `"eyes"`).
- Utilisez `""` pour désactiver la réaction pour un canal ou un compte.

## Manifeste et checklist des scopes

<AccordionGroup>
  <Accordion title="Exemple de manifeste d'application Slack">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "mpim:history",
        "users:read",
        "app_mentions:read",
        "assistant:write",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  </Accordion>

  <Accordion title="Scopes optionnels du jeton utilisateur (opérations de lecture)">
    Si vous configurez `channels.slack.userToken`, les scopes de lecture typiques sont :

    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (si vous dépendez des lectures de recherche Slack)

  </Accordion>
</AccordionGroup>

## Dépannage

<AccordionGroup>
  <Accordion title="Pas de réponses dans les canaux">
    Vérifiez, dans l'ordre :

    - `groupPolicy`
    - liste autorisée des canaux (`channels.slack.channels`)
    - `requireMention`
    - liste autorisée `users` par canal

    Commandes utiles :

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  </Accordion>

  <Accordion title="Messages privés ignorés">
    Vérifiez :

    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (ou ancien `channels.slack.dm.policy`)
    - approbations d'appairage / entrées de la liste autorisée

```bash
openclaw pairing list slack
```

  </Accordion>

  <Accordion title="Le Socket Mode ne se connecte pas">
    Validez les jetons bot + app et l'activation du Socket Mode dans les paramètres de l'application Slack.
  </Accordion>

  <Accordion title="Le mode HTTP ne reçoit pas d'événements">
    Validez :

    - le signing secret
    - le chemin webhook
    - les URL de requête Slack (Events + Interactivity + Slash Commands)
    - un `webhookPath` unique par compte HTTP

  </Accordion>

  <Accordion title="Les commandes natives/slash ne se déclenchent pas">
    Vérifiez si vous aviez l'intention d'utiliser :

    - le mode commandes natives (`channels.slack.commands.native: true`) avec les commandes slash correspondantes enregistrées dans Slack
    - ou le mode commande slash unique (`channels.slack.slashCommand.enabled: true`)

    Vérifiez également `commands.useAccessGroups` et les listes autorisées de canaux/utilisateurs.

  </Accordion>
</AccordionGroup>

## Streaming de texte

OpenClaw prend en charge le streaming de texte natif Slack via l'API Agents and AI Apps.

`channels.slack.streaming` contrôle le comportement de prévisualisation en direct :

- `off` : désactiver le streaming de prévisualisation en direct.
- `partial` (par défaut) : remplacer le texte de prévisualisation par la dernière sortie partielle.
- `block` : ajouter les mises à jour de prévisualisation par blocs.
- `progress` : afficher le texte de statut de progression pendant la génération, puis envoyer le texte final.

`channels.slack.nativeStreaming` contrôle l'API de streaming native de Slack (`chat.startStream` / `chat.appendStream` / `chat.stopStream`) lorsque `streaming` est `partial` (par défaut : `true`).

Désactiver le streaming natif Slack (conserver le comportement de prévisualisation en brouillon) :

```yaml
channels:
  slack:
    streaming: partial
    nativeStreaming: false
```

Clés héritées :

- `channels.slack.streamMode` (`replace | status_final | append`) est automatiquement migré vers `channels.slack.streaming`.
- `channels.slack.streaming` booléen est automatiquement migré vers `channels.slack.nativeStreaming`.

### Prérequis

1. Activez **Agents and AI Apps** dans les paramètres de votre application Slack.
2. Assurez-vous que l'application dispose du scope `assistant:write`.
3. Un fil de réponse doit être disponible pour ce message. La sélection du fil suit toujours `replyToMode`.

### Comportement

- Le premier morceau de texte démarre un flux (`chat.startStream`).
- Les morceaux suivants s'ajoutent au même flux (`chat.appendStream`).
- La fin de la réponse finalise le flux (`chat.stopStream`).
- Les médias et les payloads non-texte reviennent à la livraison normale.
- Si le streaming échoue en cours de réponse, OpenClaw revient à la livraison normale pour les payloads restants.

## Pointeurs de référence de configuration

Référence principale :

- [Référence de configuration - Slack](/gateway/configuration-reference#slack)

  Champs Slack importants :
  - mode/auth : `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - accès MP : `dm.enabled`, `dmPolicy`, `allowFrom` (ancien : `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - bascule de compatibilité : `dangerouslyAllowNameMatching` (mode dégradé ; gardez désactivé sauf nécessité)
  - accès canal : `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - fils/historique : `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - livraison : `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `streaming`, `nativeStreaming`
  - ops/fonctionnalités : `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Liens connexes

- [Appairage](/channels/pairing)
- [Routage des canaux](/channels/channel-routing)
- [Dépannage](/channels/troubleshooting)
- [Configuration](/gateway/configuration)
- [Commandes slash](/tools/slash-commands)
