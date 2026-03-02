---
summary: "Statut de prise en charge du bot Discord, capacités et configuration"
read_when:
  - Travail sur les fonctionnalités du canal Discord
title: "Discord"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/discord.md
  workflow: manual
---

# Discord (Bot API)

Statut : prêt pour les messages privés et les canaux de serveur via le gateway officiel Discord.

<CardGroup cols={3}>
  <Card title="Appairage" icon="link" href="/channels/pairing">
    Les messages privés Discord utilisent par défaut le mode appairage.
  </Card>
  <Card title="Commandes slash" icon="terminal" href="/tools/slash-commands">
    Comportement des commandes natives et catalogue de commandes.
  </Card>
  <Card title="Dépannage des canaux" icon="wrench" href="/channels/troubleshooting">
    Diagnostics et flux de réparation inter-canaux.
  </Card>
</CardGroup>

## Configuration rapide

Vous devrez créer une nouvelle application avec un bot, ajouter le bot a votre serveur et l'appairer a OpenClaw. Nous recommandons d'ajouter votre bot a votre propre serveur privé. Si vous n'en avez pas encore, [creez-en un d'abord](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server) (choisissez **Create My Own > For me and my friends**).

<Steps>
  <Step title="Créer une application et un bot Discord">
    Allez sur le [Portail Developpeur Discord](https://discord.com/developers/applications) et cliquez sur **New Application**. Nommez-le quelque chose comme « OpenClaw ».

    Cliquez sur **Bot** dans la barre laterale. Définissez le **Username** selon le nom que vous donnez a votre agent OpenClaw.

  </Step>

  <Step title="Activer les intents privilegies">
    Toujours sur la page **Bot**, faites defiler vers le bas jusqu'a **Privileged Gateway Intents** et activez :

    - **Message Content Intent** (requis)
    - **Server Members Intent** (recommandé ; requis pour les listes autorisées de rôles et la correspondance nom-vers-ID)
    - **Presence Intent** (optionnel ; nécessaire uniquement pour les mises a jour de presence)

  </Step>

  <Step title="Copier votre jeton de bot">
    Remontez sur la page **Bot** et cliquez sur **Reset Token**.

    <Note>
    Malgre le nom, cela généré votre premier jeton ; rien n'est « réinitialisé ».
    </Note>

    Copiez le jeton et sauvegardez-le quelque part. C'est votre **Bot Token** et vous en aurez besoin sous peu.

  </Step>

  <Step title="Générer une URL d'invitation et ajouter le bot a votre serveur">
    Cliquez sur **OAuth2** dans la barre laterale. Vous allez générer une URL d'invitation avec les bonnes permissions pour ajouter le bot a votre serveur.

    Faites defiler jusqu'a **OAuth2 URL Generator** et activez :

    - `bot`
    - `applications.commands`

    Une section **Bot Permissions** apparaitra en dessous. Activez :

    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Réactions (optionnel)

    Copiez l'URL générée en bas, collez-la dans votre navigateur, selectionnez votre serveur et cliquez sur **Continue** pour connecter. Vous devriez maintenant voir votre bot dans le serveur Discord.

  </Step>

  <Step title="Activer le mode developpeur et collecter vos identifiants">
    De retour dans l'application Discord, vous devez activer le mode developpeur pour pouvoir copier les identifiants internes.

    1. Cliquez sur **Paramètres utilisateur** (icone d'engrenage a côté de votre avatar) -> **Avance** -> activez **Mode developpeur**
    2. Faites un clic droit sur l'**icone de votre serveur** dans la barre laterale -> **Copier l'ID du serveur**
    3. Faites un clic droit sur votre **propre avatar** -> **Copier l'ID de l'utilisateur**

    Sauvegardez votre **ID de serveur** et votre **ID d'utilisateur** a côté de votre Bot Token ; vous enverrez les trois a OpenClaw a l'étape suivante.

  </Step>

  <Step title="Autoriser les messages privés des membres du serveur">
    Pour que l'appairage fonctionne, Discord doit autoriser votre bot a vous envoyer des messages privés. Faites un clic droit sur l'**icone de votre serveur** -> **Paramètres de confidentialité** -> activez **Messages directs**.

    Cela permet aux membres du serveur (y compris les bots) de vous envoyer des messages privés. Gardez ceci active si vous souhaitez utiliser les messages privés Discord avec OpenClaw. Si vous prevoyez d'utiliser uniquement les canaux de serveur, vous pouvez désactiver les messages privés après l'appairage.

  </Step>

  <Step title="Étape 0 : Définir votre jeton de bot de maniere sécurisée (ne l'envoyez pas dans le chat)">
    Votre jeton de bot Discord est un secret (comme un mot de passe). Définissez-le sur la machine executant OpenClaw avant d'envoyer un message a votre agent.

```bash
openclaw config set channels.discord.token '"YOUR_BOT_TOKEN"' --json
openclaw config set channels.discord.enabled true --json
openclaw gateway
```

    Si OpenClaw est déjà en cours d'exécution en tant que service d'arriere-plan, utilisez `openclaw gateway restart` a la place.

  </Step>

  <Step title="Configurer OpenClaw et appairer">

    <Tabs>
      <Tab title="Demander a votre agent">
        Discutez avec votre agent OpenClaw sur un canal existant (par ex. Telegram) et dites-lui. Si Discord est votre premier canal, utilisez plutot l'onglet CLI / config.

        > « J'ai déjà défini mon jeton de bot Discord dans la config. Veuillez terminer la configuration Discord avec l'ID utilisateur `<user_id>` et l'ID serveur `<server_id>`. »
      </Tab>
      <Tab title="CLI / config">
        Si vous préférez la configuration par fichier, définissez :

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

        Repli par variable d'environnement pour le compte par défaut :

```bash
DISCORD_BOT_TOKEN=...
```

      </Tab>
    </Tabs>

  </Step>

  <Step title="Approuver le premier appairage de message privé">
    Attendez que le gateway soit en cours d'exécution, puis envoyez un message privé a votre bot dans Discord. Il repondra avec un code d'appairage.

    <Tabs>
      <Tab title="Demander a votre agent">
        Envoyez le code d'appairage a votre agent sur votre canal existant :

        > « Approuve ce code d'appairage Discord : `<CODE>` »
      </Tab>
      <Tab title="CLI">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

      </Tab>
    </Tabs>

    Les codes d'appairage expirent après 1 heure.

    Vous devriez maintenant pouvoir discuter avec votre agent dans Discord via message privé.

  </Step>
</Steps>

<Note>
La résolution des jetons est sensible au compte. Les valeurs de jeton de configuration prennent le pas sur le repli par variable d'environnement. `DISCORD_BOT_TOKEN` est utilisé uniquement pour le compte par défaut.
</Note>

## Recommandé : Configurer un espace de travail de serveur

Une fois que les messages privés fonctionnent, vous pouvez configurer votre serveur Discord comme un espace de travail complet ou chaque canal obtient sa propre session d'agent avec son propre contexte. C'est recommandé pour les serveurs privés ou il n'y a que vous et votre bot.

<Steps>
  <Step title="Ajouter votre serveur a la liste autorisée des serveurs">
    Cela permet a votre agent de répondre dans n'importe quel canal de votre serveur, pas seulement les messages privés.

    <Tabs>
      <Tab title="Demander a votre agent">
        > « Ajouté mon ID de serveur Discord `<server_id>` a la liste autorisée des serveurs »
      </Tab>
      <Tab title="Config">

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: true,
          users: ["YOUR_USER_ID"],
        },
      },
    },
  },
}
```

      </Tab>
    </Tabs>

  </Step>

  <Step title="Autoriser les réponses sans @mention">
    Par défaut, votre agent ne répond dans les canaux de serveur que lorsqu'il est @mentionne. Pour un serveur privé, vous voudrez probablement qu'il reponde a chaque message.

    <Tabs>
      <Tab title="Demander a votre agent">
        > « Autorisé mon agent a répondre sur ce serveur sans avoir a être @mentionne »
      </Tab>
      <Tab title="Config">
        Définissez `requireMention: false` dans votre configuration de serveur :

```json5
{
  channels: {
    discord: {
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: false,
        },
      },
    },
  },
}
```

      </Tab>
    </Tabs>

  </Step>

  <Step title="Planifier la mémoire dans les canaux de serveur">
    Par défaut, la mémoire a long terme (MEMORY.md) ne se charge que dans les sessions de messages privés. Les canaux de serveur ne chargent pas automatiquement MEMORY.md.

    <Tabs>
      <Tab title="Demander a votre agent">
        > « Quand je pose des questions dans les canaux Discord, utilisé memory_search ou memory_get si tu as besoin de contexte a long terme depuis MEMORY.md. »
      </Tab>
      <Tab title="Manuel">
        Si vous avez besoin de contexte partage dans chaque canal, mettez les instructions stables dans `AGENTS.md` ou `USER.md` (ils sont injectés pour chaque session). Gardez les notes a long terme dans `MEMORY.md` et accedez-y a la demande avec les outils de mémoire.
      </Tab>
    </Tabs>

  </Step>
</Steps>

Maintenant creez quelques canaux sur votre serveur Discord et commencez a discuter. Votre agent peut voir le nom du canal, et chaque canal obtient sa propre session isolee, donc vous pouvez configurer `#coding`, `#home`, `#research`, ou tout ce qui convient a votre flux de travail.

## Modèle d'exécution

- Le Gateway possède la connexion Discord.
- Le routage des réponses est déterministe : les messages entrants Discord répondent vers Discord.
- Par défaut (`session.dmScope=main`), les discussions directes partagent la session principale de l'agent (`agent:main:main`).
- Les canaux de serveur sont des clés de session isolees (`agent:<agentId>:discord:channel:<channelId>`).
- Les messages privés de groupe sont ignorés par défaut (`channels.discord.dm.groupEnabled=false`).
- Les commandes slash natives s'executent dans des sessions de commande isolees (`agent:<agentId>:discord:slash:<userId>`), tout en portant `CommandTargetSessionKey` vers la session de conversation acheminee.

## Canaux de forum

Les canaux de forum et de médias Discord n'acceptent que les publications de fil. OpenClaw prend en charge deux facons de les créer :

- Envoyer un message au parent du forum (`channel:<forumId>`) pour créer automatiquement un fil. Le titre du fil utilise la première ligne non vide de votre message.
- Utiliser `openclaw message thread create` pour créer un fil directement. Ne passez pas `--message-id` pour les canaux de forum.

Exemple : envoyer au parent du forum pour créer un fil

```bash
openclaw message send --channel discord --target channel:<forumId> \
  --message "Topic title\nBody of the post"
```

Exemple : créer un fil de forum explicitement

```bash
openclaw message thread create --channel discord --target channel:<forumId> \
  --thread-name "Topic title" --message "Body of the post"
```

Les parents de forum n'acceptent pas les composants Discord. Si vous avez besoin de composants, envoyez au fil lui-même (`channel:<threadId>`).

## Composants interactifs

OpenClaw prend en charge les conteneurs de composants v2 Discord pour les messages d'agent. Utilisez l'outil de message avec un payload `components`. Les résultats d'interaction sont renvoyes a l'agent comme des messages entrants normaux et suivent les paramètres `replyToMode` Discord existants.

Blocs pris en charge :

- `text`, `section`, `separator`, `actions`, `média-gallery`, `file`
- Les rangees d'actions autorisent jusqu'a 5 boutons ou un seul menu de sélection
- Types de sélection : `string`, `user`, `rôle`, `mentionable`, `channel`

Par défaut, les composants sont a usage unique. Définissez `components.reusable=true` pour permettre aux boutons, sélections et formulaires d'être utilisés plusieurs fois jusqu'a leur expiration.

Pour restreindre qui peut cliquer sur un bouton, définissez `allowedUsers` sur ce bouton (identifiants utilisateur Discord, tags ou `*`). Lorsque configuré, les utilisateurs non correspondants reçoivent un refus ephemere.

Les commandes slash `/model` et `/models` ouvrent un sélecteur de modèle interactif avec des listes deroulantes de fournisseur et de modèle plus une étape de soumission. La réponse du sélecteur est ephemere et seul l'utilisateur invoquant peut l'utiliser.

Pieces jointes :

- Les blocs `file` doivent pointer vers une référence de piece jointe (`attachment://<filename>`)
- Fournissez la piece jointe via `média`/`path`/`filePath` (fichier unique) ; utilisez `média-gallery` pour plusieurs fichiers
- Utilisez `filename` pour remplacer le nom de téléchargement lorsqu'il doit correspondre a la référence de piece jointe

Formulaires modaux :

- Ajoutez `components.modal` avec jusqu'a 5 champs
- Types de champs : `text`, `checkbox`, `radio`, `select`, `rôle-select`, `user-select`
- OpenClaw ajoute automatiquement un bouton declencheur

Exemple :

```json5
{
  channel: "discord",
  action: "send",
  to: "channel:123456789012345678",
  message: "Optional fallback text",
  components: {
    reusable: true,
    text: "Choose a path",
    blocks: [
      {
        type: "actions",
        buttons: [
          {
            label: "Approve",
            style: "success",
            allowedUsers: ["123456789012345678"],
          },
          { label: "Decline", style: "danger" },
        ],
      },
      {
        type: "actions",
        select: {
          type: "string",
          placeholder: "Pick an option",
          options: [
            { label: "Option A", value: "a" },
            { label: "Option B", value: "b" },
          ],
        },
      },
    ],
    modal: {
      title: "Details",
      triggerLabel: "Open form",
      fields: [
        { type: "text", label: "Requester" },
        {
          type: "select",
          label: "Priority",
          options: [
            { label: "Low", value: "low" },
            { label: "High", value: "high" },
          ],
        },
      ],
    },
  },
}
```

## Contrôle d'accès et routage

<Tabs>
  <Tab title="Politique de messages privés">
    `channels.discord.dmPolicy` contrôle l'accès aux messages privés (ancien : `channels.discord.dm.policy`) :

    - `pairing` (par défaut)
    - `allowlist`
    - `open` (nécessite que `channels.discord.allowFrom` inclue `"*"` ; ancien : `channels.discord.dm.allowFrom`)
    - `disabled`

    Si la politique de messages privés n'est pas ouverte, les utilisateurs inconnus sont bloqués (ou invites a l'appairage en mode `pairing`).

    Precedence multi-comptes :

    - `channels.discord.accounts.default.allowFrom` s'applique uniquement au compte `default`.
    - Les comptes nommes héritent de `channels.discord.allowFrom` lorsque leur propre `allowFrom` n'est pas défini.
    - Les comptes nommes n'héritent pas de `channels.discord.accounts.default.allowFrom`.

    Format de cible de messages privés pour la livraison :

    - `user:<id>`
    - mention `<@id>`

    Les identifiants numériques seuls sont ambigus et rejetes sauf si un type explicite de cible utilisateur/canal est fourni.

  </Tab>

  <Tab title="Politique de serveur">
    La gestion des serveurs est controlee par `channels.discord.groupPolicy` :

    - `open`
    - `allowlist`
    - `disabled`

    La base sécurisée lorsque `channels.discord` existe est `allowlist`.

    Comportement `allowlist` :

    - le serveur doit correspondre a `channels.discord.guilds` (l'`id` est préféré, le slug accepte)
    - listes autorisées d'expéditeurs optionnelles : `users` (identifiants stables recommandes) et `rôles` (identifiants de rôle uniquement) ; si l'un est configuré, les expéditeurs sont autorisés lorsqu'ils correspondent a `users` OU `rôles`
    - la correspondance directe nom/tag est désactivée par défaut ; activez `channels.discord.dangerouslyAllowNameMatching: true` uniquement comme mode de compatibilité d'urgence
    - les noms/tags sont pris en charge pour `users`, mais les identifiants sont plus surs ; `openclaw security audit` avertit lorsque des entrées nom/tag sont utilisées
    - si un serveur a des `channels` configurés, les canaux non listes sont refuses
    - si un serveur n'a pas de bloc `channels`, tous les canaux de ce serveur autorisé sont permis

    Exemple :

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

    Si vous définissez uniquement `DISCORD_BOT_TOKEN` et ne creez pas de bloc `channels.discord`, le repli a l'exécution est `groupPolicy="allowlist"` (avec un avertissement dans les logs), même si `channels.defaults.groupPolicy` est `open`.

  </Tab>

  <Tab title="Mentions et messages privés de groupe">
    Les messages de serveur sont filtrés par mention par défaut.

    La détection de mention inclut :

    - la mention explicite du bot
    - les modèles de mention configurés (`agents.list[].groupChat.mentionPatterns`, repli `messages.groupChat.mentionPatterns`)
    - le comportement implicite de réponse au bot dans les cas pris en charge

    `requireMention` est configuré par serveur/canal (`channels.discord.guilds...`).

    Messages privés de groupe :

    - par défaut : ignorés (`dm.groupEnabled=false`)
    - liste autorisée optionnelle via `dm.groupChannels` (identifiants de canal ou slugs)

  </Tab>
</Tabs>

### Routage d'agent base sur les rôles

Utilisez `bindings[].match.rôles` pour acheminer les membres de serveur Discord vers differents agents par identifiant de rôle. Les liaisons basees sur les rôles n'acceptent que les identifiants de rôle et sont evaluees après les liaisons peer ou parent-peer et avant les liaisons serveur uniquement. Si une liaison définit également d'autres champs de correspondance (par exemple `peer` + `guildId` + `rôles`), tous les champs configurés doivent correspondre.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Configuration du Portail Developpeur

<AccordionGroup>
  <Accordion title="Créer l'application et le bot">

    1. Portail Developpeur Discord -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. Copier le jeton du bot

  </Accordion>

  <Accordion title="Intents privilegies">
    Dans **Bot -> Privileged Gateway Intents**, activez :

    - Message Content Intent
    - Server Members Intent (recommandé)

    L'intent de presence est optionnel et n'est requis que si vous souhaitez recevoir des mises a jour de presence. Définir la presence du bot (`setPresence`) ne nécessite pas d'activer les mises a jour de presence pour les membres.

  </Accordion>

  <Accordion title="Scopes OAuth et permissions de base">
    Generateur d'URL OAuth :

    - scopes : `bot`, `applications.commands`

    Permissions de base typiques :

    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Réactions (optionnel)

    Evitez `Administrator` sauf si explicitement nécessaire.

  </Accordion>

  <Accordion title="Copier les identifiants">
    Activez le mode developpeur Discord, puis copiez :

    - identifiant du serveur
    - identifiant du canal
    - identifiant de l'utilisateur

    Préférez les identifiants numériques dans la configuration OpenClaw pour des audits et sondes fiables.

  </Accordion>
</AccordionGroup>

## Commandes natives et autorisation de commandes

- `commands.native` est par défaut `"auto"` et est activé pour Discord.
- Remplacement par canal : `channels.discord.commands.native`.
- `commands.native=false` efface explicitement les commandes natives Discord precedemment enregistrees.
- L'autorisation des commandes natives utilisé les mêmes listes autorisées/politiques Discord que la gestion normale des messages.
- Les commandes peuvent toujours être visibles dans l'interface Discord pour les utilisateurs non autorisés ; l'exécution applique toujours l'authentification OpenClaw et renvoie « not authorized ».

Voir [Commandes slash](/tools/slash-commands) pour le catalogue et le comportement des commandes.

Paramètres par défaut des commandes slash :

- `ephemeral: true`

## Details des fonctionnalités

<AccordionGroup>
  <Accordion title="Balises de réponse et réponses natives">
    Discord prend en charge les balises de réponse dans la sortie de l'agent :

    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`

    Contrôle par `channels.discord.replyToMode` :

    - `off` (par défaut)
    - `first`
    - `all`

    Note : `off` désactive le fil de réponse implicite. Les balises explicites `[[reply_to_*]]` sont toujours honorees.

    Les identifiants de message sont exposes dans le contexte/historique pour que les agents puissent cibler des messages spécifiques.

  </Accordion>

  <Accordion title="Aperçu en direct du flux">
    OpenClaw peut diffuser des brouillons de réponse en envoyant un message temporaire et en le modifiant au fur et a mesure que le texte arrive.

    - `channels.discord.streaming` contrôle la diffusion d'aperçu (`off` | `partial` | `block` | `progress`, par défaut : `off`).
    - `progress` est accepte pour la coherence inter-canal et correspond a `partial` sur Discord.
    - `channels.discord.streamMode` est un alias historique et est auto-migre.
    - `partial` modifie un seul message d'aperçu au fur et a mesure que les tokens arrivent.
    - `block` émet des morceaux de taille brouillon (utilisez `draftChunk` pour ajuster la taille et les points de coupure).

    Exemple :

```json5
{
  channels: {
    discord: {
      streaming: "partial",
    },
  },
}
```

    Valeurs par défaut du découpage en mode `block` (limitées a `channels.discord.textChunkLimit`) :

```json5
{
  channels: {
    discord: {
      streaming: "block",
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph",
      },
    },
  },
}
```

    La diffusion d'aperçu est textuelle uniquement ; les réponses médias se rabattent sur la livraison normale.

    Note : la diffusion d'aperçu est séparée de la diffusion par blocs. Lorsque la diffusion par blocs est explicitement
    activée pour Discord, OpenClaw saute le flux d'aperçu pour eviter la double diffusion.

  </Accordion>

  <Accordion title="Historique, contexte et comportement des fils">
    Contexte d'historique du serveur :

    - `channels.discord.historyLimit` par défaut `20`
    - repli : `messages.groupChat.historyLimit`
    - `0` désactive

    Contrôles de l'historique des messages privés :

    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`

    Comportement des fils :

    - Les fils Discord sont achemines comme des sessions de canal
    - Les métadonnées du fil parent peuvent être utilisées pour le lien de session parent
    - La configuration du fil hérité de la configuration du canal parent sauf si une entrée spécifique au fil existe

    Les sujets de canal sont injectés comme contexte **non fiable** (pas comme prompt système).

  </Accordion>

  <Accordion title="Sessions liées aux fils pour les sous-agents">
    Discord peut lier un fil a une cible de session pour que les messages de suivi dans ce fil continuent a être achemines vers la même session (y compris les sessions de sous-agents).

    Commandes :

    - `/focus <target>` lier le fil actuel/nouveau a une cible de sous-agent/session
    - `/unfocus` supprimer la liaison du fil actuel
    - `/agents` afficher les executions activés et l'état de liaison
    - `/session ttl <duration|off>` inspecter/mettre a jour le TTL de defocalisation automatique pour les liaisons focalisees

    Configuration :

```json5
{
  session: {
    threadBindings: {
      enabled: true,
      ttlHours: 24,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        ttlHours: 24,
        spawnSubagentSessions: false, // opt-in
      },
    },
  },
}
```

    Notes :

    - `session.threadBindings.*` définit les valeurs par défaut globales.
    - `channels.discord.threadBindings.*` remplace le comportement Discord.
    - `spawnSubagentSessions` doit être true pour créer/lier automatiquement des fils pour `sessions_spawn({ thread: true })`.
    - `spawnAcpSessions` doit être true pour créer/lier automatiquement des fils pour ACP (`/acp spawn ... --thread ...` ou `sessions_spawn({ runtime: "acp", thread: true })`).
    - Si les liaisons de fil sont désactivées pour un compte, `/focus` et les opérations de liaison de fil associées ne sont pas disponibles.

    Voir [Sous-agents](/tools/subagents), [Agents ACP](/tools/acp-agents) et [Référence de configuration](/gateway/configuration-reference).

  </Accordion>

  <Accordion title="Notifications de reaction">
    Mode de notification de reaction par serveur :

    - `off`
    - `own` (par défaut)
    - `all`
    - `allowlist` (utilisé `guilds.<id>.users`)

    Les événements de reaction sont transformes en événements système et attaches a la session Discord acheminee.

  </Accordion>

  <Accordion title="Réactions d'accusé de reception">
    `ackReaction` envoie un emoji d'accusé de reception pendant qu'OpenClaw traite un message entrant.

    Ordre de résolution :

    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - repli sur l'emoji d'identité de l'agent (`agents.list[].identity.emoji`, sinon "👀")

    Notes :

    - Discord accepte les emojis unicode ou les noms d'emojis personnalisés.
    - Utilisez `""` pour désactiver la reaction pour un canal ou un compte.

  </Accordion>

  <Accordion title="Écritures de configuration">
    Les écritures de configuration initiees par le canal sont activées par défaut.

    Cela affecte les flux `/config set|unset` (lorsque les fonctionnalités de commande sont activées).

    Désactiver :

```json5
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

  </Accordion>

  <Accordion title="Proxy du Gateway">
    Acheminez le trafic WebSocket du gateway Discord et les recherches REST de démarrage (identifiant d'application + résolution de liste autorisée) via un proxy HTTP(S) avec `channels.discord.proxy`.

```json5
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

    Remplacement par compte :

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  </Accordion>

  <Accordion title="Support PluralKit">
    Activez la résolution PluralKit pour mapper les messages proxyes a l'identité du membre du système :

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optionnel ; necessaire pour les systemes prives
      },
    },
  },
}
```

    Notes :

    - les listes autorisées peuvent utiliser `pk:<memberId>`
    - les noms d'affichage des membres sont mis en correspondance par nom/slug uniquement lorsque `channels.discord.dangerouslyAllowNameMatching: true`
    - les recherches utilisent l'identifiant du message original et sont contraintes par fenêtre temporelle
    - si la recherche échoue, les messages proxyes sont traites comme des messages de bot et ignorés sauf si `allowBots=true`

  </Accordion>

  <Accordion title="Configuration de la presence">
    Les mises a jour de presence ne sont appliquées que lorsque vous définissez un champ de statut ou d'activité.

    Exemple de statut uniquement :

```json5
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

    Exemple d'activité (le statut personnalisé est le type d'activité par défaut) :

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    Exemple de streaming :

```json5
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

    Carte des types d'activité :

    - 0 : Playing
    - 1 : Streaming (nécessite `activityUrl`)
    - 2 : Listening
    - 3 : Watching
    - 4 : Custom (utilisé le texte d'activité comme état du statut ; l'emoji est optionnel)
    - 5 : Competing

  </Accordion>

  <Accordion title="Approbations d'exécution dans Discord">
    Discord prend en charge les approbations d'exécution par bouton dans les messages privés et peut optionnellement publier les invites d'approbation dans le canal d'origine.

    Chemin de configuration :

    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, par défaut : `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`

    Lorsque `target` est `channel` ou `both`, l'invite d'approbation est visible dans le canal. Seuls les approbateurs configurés peuvent utiliser les boutons ; les autres utilisateurs reçoivent un refus ephemere. Les invites d'approbation incluent le texte de la commande, donc n'activez la livraison par canal que dans les canaux de confiance. Si l'identifiant du canal ne peut pas être derive de la clé de session, OpenClaw se rabat sur la livraison par message privé.

    Si les approbations échouent avec des identifiants d'approbation inconnus, verifiez la liste des approbateurs et l'activation de la fonctionnalité.

    Documentation associée : [Approbations d'exécution](/tools/exec-approvals)

  </Accordion>
</AccordionGroup>

## Outils et contrôles d'action

Les actions de message Discord incluent la messagerie, l'administration de canal, la moderation, la presence et les actions de métadonnées.

Exemples principaux :

- messagerie : `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- reactions : `react`, `reactions`, `emojiList`
- moderation : `timeout`, `kick`, `ban`
- presence : `setPresence`

Les contrôles d'action se trouvent sous `channels.discord.actions.*`.

Comportement par défaut des contrôles :

| Groupe d'actions                                                                                                                                                             | Par défaut |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | activé   |
| rôles                                                                                                                                                                    | désactivé |
| moderation                                                                                                                                                               | désactivé |
| presence                                                                                                                                                                 | désactivé |

## Interface Composants v2

OpenClaw utilise les composants v2 Discord pour les approbations d'exécution et les marqueurs inter-contexte. Les actions de message Discord peuvent également accepter des `components` pour une interface personnalisée (avance ; nécessite des instances de composants Carbon), tandis que les anciens `embeds` restent disponibles mais ne sont pas recommandes.

- `channels.discord.ui.components.accentColor` définit la couleur d'accentuation utilisée par les conteneurs de composants Discord (hex).
- Définir par compte avec `channels.discord.accounts.<id>.ui.components.accentColor`.
- Les `embeds` sont ignorés lorsque les composants v2 sont présents.

Exemple :

```json5
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## Canaux vocaux

OpenClaw peut rejoindre les canaux vocaux Discord pour des conversations en temps reel et continues. Ceci est distinct des pieces jointes de messages vocaux.

Prerequis :

- Activez les commandes natives (`commands.native` ou `channels.discord.commands.native`).
- Configurez `channels.discord.voice`.
- Le bot a besoin des permissions Connect + Speak dans le canal vocal cible.

Utilisez la commande native Discord uniquement `/vc join|leave|status` pour contrôler les sessions. La commande utilise l'agent par défaut du compte et suit les mêmes règles de liste autorisée et de politique de groupe que les autres commandes Discord.

Exemple de connexion automatique :

```json5
{
  channels: {
    discord: {
      voice: {
        enabled: true,
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        tts: {
          provider: "openai",
          openai: { voice: "alloy" },
        },
      },
    },
  },
}
```

Notes :

- `voice.tts` remplace `messages.tts` pour la lecture vocale uniquement.
- Le vocal est activé par défaut ; définissez `channels.discord.voice.enabled=false` pour le désactiver.
- `voice.daveEncryption` et `voice.decryptionFailureTolerance` sont transmis aux options de connexion `@discordjs/voice`.
- Les valeurs par défaut de `@discordjs/voice` sont `daveEncryption=true` et `decryptionFailureTolerance=24` si non définis.
- OpenClaw surveille également les échecs de dechiffrement en reception et se retablit automatiquement en quittant/rejoignant le canal vocal après des échecs repetes dans une courte fenêtre.
- Si les logs de reception montrent régulièrement `DecryptionFailed(UnencryptedWhenPassthroughDisabled)`, il peut s'agir du bug de reception `@discordjs/voice` en amont suivi dans [discord.js #11419](https://github.com/discordjs/discord.js/issues/11419).

## Messages vocaux

Les messages vocaux Discord affichent un aperçu de forme d'onde et nécessitent un audio OGG/Opus plus des métadonnées. OpenClaw généré la forme d'onde automatiquement, mais il a besoin de `ffmpeg` et `ffprobe` disponibles sur l'hôte du gateway pour inspecter et convertir les fichiers audio.

Prerequis et contraintes :

- Fournissez un **chemin de fichier local** (les URL sont rejetees).
- Omettez le contenu texte (Discord n'autorisé pas texte + message vocal dans le même payload).
- Tout format audio est accepte ; OpenClaw convertit en OGG/Opus si nécessaire.

Exemple :

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Dépannage

<AccordionGroup>
  <Accordion title="Intents non autorisés utilisés ou le bot ne voit pas les messages du serveur">

    - activez Message Content Intent
    - activez Server Members Intent lorsque vous dependez de la résolution utilisateur/membre
    - redemarrez le gateway après avoir change les intents

  </Accordion>

  <Accordion title="Messages du serveur bloqués de maniere inattendue">

    - verifiez `groupPolicy`
    - verifiez la liste autorisée du serveur sous `channels.discord.guilds`
    - si la carte `channels` du serveur existe, seuls les canaux listes sont autorisés
    - verifiez le comportement `requireMention` et les modèles de mention

    Vérifications utiles :

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  </Accordion>

  <Accordion title="Require mention false mais toujours bloque">
    Causes courantes :

    - `groupPolicy="allowlist"` sans liste autorisée de serveur/canal correspondante
    - `requireMention` configuré au mauvais endroit (doit être sous `channels.discord.guilds` ou entrée de canal)
    - expéditeur bloque par la liste autorisée `users` du serveur/canal

  </Accordion>

  <Accordion title="Incoherences d'audit de permissions">
    Les vérifications de permissions `channels status --probe` ne fonctionnent que pour les identifiants de canal numériques.

    Si vous utilisez des clés slug, la correspondance a l'exécution peut toujours fonctionner, mais la sonde ne peut pas vérifier complètement les permissions.

  </Accordion>

  <Accordion title="Problèmes de messages privés et d'appairage">

    - messages privés désactivés : `channels.discord.dm.enabled=false`
    - politique de messages privés désactivée : `channels.discord.dmPolicy="disabled"` (ancien : `channels.discord.dm.policy`)
    - en attente d'approbation d'appairage en mode `pairing`

  </Accordion>

  <Accordion title="Boucles bot a bot">
    Par défaut, les messages écrits par des bots sont ignorés.

    Si vous définissez `channels.discord.allowBots=true`, utilisez des règles strictes de mention et de liste autorisée pour eviter les comportements de boucle.

  </Accordion>

  <Accordion title="Coupures STT vocales avec DecryptionFailed(...)">

    - maintenez OpenClaw a jour (`openclaw update`) pour que la logique de récupération de reception vocale Discord soit présenté
    - confirmez `channels.discord.voice.daveEncryption=true` (par défaut)
    - commencez avec `channels.discord.voice.decryptionFailureTolerance=24` (valeur par défaut en amont) et ajustez seulement si nécessaire
    - surveillez les logs pour :
      - `discord voice: DAVE decrypt failures detected`
      - `discord voice: repeated decrypt failures; attempting rejoin`
    - si les échecs persistent après la reconnexion automatique, collectez les logs et comparez avec [discord.js #11419](https://github.com/discordjs/discord.js/issues/11419)

  </Accordion>
</AccordionGroup>

## Pointeurs de référence de configuration

Référence principale :

- [Référence de configuration - Discord](/gateway/configuration-reference#discord)

Champs Discord a fort signal :

- démarrage/auth : `enabled`, `token`, `accounts.*`, `allowBots`
- politique : `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- commande : `commands.native`, `commands.useAccessGroups`, `configWrites`, `slashCommand.*`
- réponse/historique : `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- livraison : `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- streaming : `streaming` (alias historique : `streamMode`), `draftChunk`, `blockStreaming`, `blockStreamingCoalesce`
- média/nouvelle tentative : `mediaMaxMb`, `retry`
- actions : `actions.*`
- presence : `activity`, `status`, `activityType`, `activityUrl`
- interface : `ui.components.accentColor`
- fonctionnalités : `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Sécurité et opérations

- Traitez les jetons de bot comme des secrets (`DISCORD_BOT_TOKEN` préféré dans les environnements supervises).
- Accordez les permissions Discord les moins privilegiees.
- Si le déploiement/l'état des commandes est obsolete, redemarrez le gateway et re-verifiez avec `openclaw channels status --probe`.

## Liens associés

- [Appairage](/channels/pairing)
- [Routage des canaux](/channels/channel-routing)
- [Routage multi-agent](/concepts/multi-agent)
- [Dépannage](/channels/troubleshooting)
- [Commandes slash](/tools/slash-commands)
