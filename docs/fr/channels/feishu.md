---
summary: "Présentation du bot Feishu, fonctionnalités et configuration"
read_when:
  - Vous souhaitez connecter un bot Feishu/Lark
  - Vous configurez le canal Feishu
title: Feishu
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/feishu.md
  workflow: manual
---

# Bot Feishu

Feishu (Lark) est une plateforme de chat d'équipe utilisée par les entreprises pour la messagerie et la collaboration. Ce plugin connecté OpenClaw à un bot Feishu/Lark en utilisant l'abonnement aux événements WebSocket de la plateforme, ce qui permet de recevoir des messages sans exposer d'URL webhook publique.

---

## Plugin requis

Installez le plugin Feishu :

```bash
openclaw plugins install @openclaw/feishu
```

Checkout local (depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/feishu
```

---

## Démarrage rapide

Il existe deux manières d'ajouter le canal Feishu :

### Méthode 1 : assistant de configuration (recommandé)

Si vous venez d'installer OpenClaw, lancez l'assistant :

```bash
openclaw onboard
```

L'assistant vous guide à travers :

1. La création d'une application Feishu et la collecte des identifiants
2. La configuration des identifiants dans OpenClaw
3. Le démarrage du Gateway

Après la configuration, vérifiez le statut du Gateway :

- `openclaw gateway status`
- `openclaw logs --follow`

### Méthode 2 : configuration par CLI

Si vous avez déjà terminé l'installation initiale, ajoutez le canal via la CLI :

```bash
openclaw channels add
```

Choisissez **Feishu**, puis entrez l'App ID et l'App Secret.

Après la configuration, gérez le Gateway :

- `openclaw gateway status`
- `openclaw gateway restart`
- `openclaw logs --follow`

---

## Étape 1 : Créer une application Feishu

### 1. Ouvrir Feishu Open Platform

Visitez [Feishu Open Platform](https://open.feishu.cn/app) et connectez-vous.

Les tenants Lark (global) doivent utiliser [https://open.larksuite.com/app](https://open.larksuite.com/app) et définir `domain: "lark"` dans la configuration Feishu.

### 2. Créer une application

1. Cliquez sur **Create enterprise app**
2. Remplissez le nom et la description de l'application
3. Choisissez une icône d'application

![Create enterprise app](../images/feishu-step2-create-app.png)

### 3. Copier les identifiants

Depuis **Credentials & Basic Info**, copiez :

- **App ID** (format : `cli_xxx`)
- **App Secret**

**Important :** gardez l'App Secret confidentiel.

![Get credentials](../images/feishu-step3-credentials.png)

### 4. Configurer les permissions

Dans **Permissions**, cliquez sur **Batch import** et collez :

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![Configuré permissions](../images/feishu-step4-permissions.png)

### 5. Activer la capacité bot

Dans **App Capability** > **Bot** :

1. Activez la capacité bot
2. Définissez le nom du bot

![Enable bot capability](../images/feishu-step5-bot-capability.png)

### 6. Configurer l'abonnement aux événements

**Important :** avant de configurer l'abonnement aux événements, assurez-vous :

1. D'avoir déjà exécuté `openclaw channels add` pour Feishu
2. Que le Gateway est en cours d'exécution (`openclaw gateway status`)

Dans **Event Subscription** :

1. Choisissez **Use long connection to receive events** (WebSocket)
2. Ajoutez l'événement : `im.message.receive_v1`

Si le Gateway n'est pas en cours d'exécution, la configuration de connexion longue peut échouer à l'enregistrement.

![Configuré event subscription](../images/feishu-step6-event-subscription.png)

### 7. Publier l'application

1. Créez une version dans **Version Management & Release**
2. Soumettez pour révision et publiez
3. Attendez l'approbation de l'administrateur (les applications d'entreprise s'approuvent généralement automatiquement)

---

## Étape 2 : Configurer OpenClaw

### Configurer avec l'assistant (recommandé)

```bash
openclaw channels add
```

Choisissez **Feishu** et collez votre App ID + App Secret.

### Configurer via le fichier de configuration

Éditez `~/.openclaw/openclaw.json` :

```json5
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "My AI assistant",
        },
      },
    },
  },
}
```

Si vous utilisez `connectionMode: "webhook"`, définissez `verificationToken`. Le serveur webhook Feishu écoute sur `127.0.0.1` par défaut ; définissez `webhookHost` uniquement si vous avez intentionnellement besoin d'une adresse d'écoute différente.

### Configurer via les variables d'environnement

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Domaine Lark (global)

Si votre tenant est sur Lark (international), définissez le domaine sur `lark` (ou une chaîne de domaine complète). Vous pouvez le définir dans `channels.feishu.domain` ou par compte (`channels.feishu.accounts.<id>.domain`).

```json5
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

---

## Étape 3 : Démarrer et tester

### 1. Démarrer le Gateway

```bash
openclaw gateway
```

### 2. Envoyer un message de test

Dans Feishu, trouvez votre bot et envoyez un message.

### 3. Approuver l'appairage

Par défaut, le bot répond avec un code d'appairage. Approuvez-le :

```bash
openclaw pairing approve feishu <CODE>
```

Après approbation, vous pouvez discuter normalement.

---

## Aperçu

- **Canal bot Feishu** : bot Feishu géré par le Gateway
- **Routage déterministe** : les réponses retournent toujours vers Feishu
- **Isolation des sessions** : les MP partagent une session principale ; les groupes sont isolés
- **Connexion WebSocket** : connexion longue via le SDK Feishu, aucune URL publique nécessaire

---

## Contrôle d'accès

### Messages privés

- **Par défaut** : `dmPolicy: "pairing"` (les utilisateurs inconnus reçoivent un code d'appairage)
- **Approuver l'appairage** :

  ```bash
  openclaw pairing list feishu
  openclaw pairing approve feishu <CODE>
  ```

- **Mode liste autorisée** : définissez `channels.feishu.allowFrom` avec les Open ID autorisés

### Discussions de groupe

**1. Politique de groupe** (`channels.feishu.groupPolicy`) :

- `"open"` = autoriser tout le monde dans les groupes (par défaut)
- `"allowlist"` = autoriser uniquement `groupAllowFrom`
- `"disabled"` = désactiver les messages de groupe

**2. Exigence de mention** (`channels.feishu.groups.<chat_id>.requireMention`) :

- `true` = nécessite une @mention (par défaut)
- `false` = répondre sans mentions

---

## Exemples de configuration de groupe

### Autoriser tous les groupes, exiger @mention (par défaut)

```json5
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // requireMention par défaut : true
    },
  },
}
```

### Autoriser tous les groupes, pas de @mention requis

```json5
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### Autoriser uniquement certains utilisateurs dans les groupes

```json5
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["ou_xxx", "ou_yyy"],
    },
  },
}
```

---

## Obtenir les ID de groupe/utilisateur

### ID de groupe (chat_id)

Les ID de groupe ressemblent à `oc_xxx`.

**Méthode 1 (recommandée)**

1. Démarrez le Gateway et @mentionnez le bot dans le groupe
2. Exécutez `openclaw logs --follow` et cherchez `chat_id`

**Méthode 2**

Utilisez le débogueur d'API Feishu pour lister les discussions de groupe.

### ID utilisateur (open_id)

Les ID utilisateur ressemblent à `ou_xxx`.

**Méthode 1 (recommandée)**

1. Démarrez le Gateway et envoyez un MP au bot
2. Exécutez `openclaw logs --follow` et cherchez `open_id`

**Méthode 2**

Vérifiez les demandes d'appairage pour les Open ID des utilisateurs :

```bash
openclaw pairing list feishu
```

---

## Commandes courantes

| Commande  | Description            |
| --------- | ---------------------- |
| `/status` | Afficher le statut du bot |
| `/reset`  | Réinitialiser la session  |
| `/model`  | Afficher/changer le modèle |

> Note : Feishu ne prend pas encore en charge les menus de commandes natives, les commandes doivent donc être envoyées en texte.

## Commandes de gestion du Gateway

| Commande                     | Description                       |
| ---------------------------- | --------------------------------- |
| `openclaw gateway status`    | Afficher le statut du Gateway     |
| `openclaw gateway install`   | Installer/démarrer le service     |
| `openclaw gateway stop`      | Arrêter le service                |
| `openclaw gateway restart`   | Redémarrer le service             |
| `openclaw logs --follow`     | Suivre les logs du Gateway        |

---

## Dépannage

### Le bot ne répond pas dans les discussions de groupe

1. Assurez-vous que le bot est ajouté au groupe
2. Assurez-vous de @mentionner le bot (comportement par défaut)
3. Vérifiez que `groupPolicy` n'est pas défini sur `"disabled"`
4. Vérifiez les logs : `openclaw logs --follow`

### Le bot ne reçoit pas de messages

1. Assurez-vous que l'application est publiée et approuvée
2. Assurez-vous que l'abonnement aux événements inclut `im.message.receive_v1`
3. Assurez-vous que la **connexion longue** est activée
4. Assurez-vous que les permissions de l'application sont complètes
5. Assurez-vous que le Gateway est en cours d'exécution : `openclaw gateway status`
6. Vérifiez les logs : `openclaw logs --follow`

### Fuite de l'App Secret

1. Réinitialisez l'App Secret dans Feishu Open Platform
2. Mettez à jour l'App Secret dans votre configuration
3. Redémarrez le Gateway

### Échecs d'envoi de messages

1. Assurez-vous que l'application a la permission `im:message:send_as_bot`
2. Assurez-vous que l'application est publiée
3. Vérifiez les logs pour les erreurs détaillées

---

## Configuration avancée

### Comptes multiples

```json5
{
  channels: {
    feishu: {
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Primary bot",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

### Limités de messages

- `textChunkLimit` : taille des morceaux de texte sortant (par défaut : 2000 caractères)
- `mediaMaxMb` : limité d'upload/download de médias (par défaut : 30 Mo)

### Streaming

Feishu prend en charge les réponses en streaming via des cartes interactives. Lorsque cette option est activée, le bot met à jour une carte au fur et à mesure de la génération du texte.

```json5
{
  channels: {
    feishu: {
      streaming: true, // activer la sortie en carte streaming (par défaut true)
      blockStreaming: true, // activer le streaming par blocs (par défaut true)
    },
  },
}
```

Définissez `streaming: false` pour attendre la réponse complète avant l'envoi.

### Routage multi-agent

Utilisez `bindings` pour router les MP ou groupes Feishu vers différents agents.

```json5
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

Champs de routage :

- `match.channel` : `"feishu"`
- `match.peer.kind` : `"direct"` ou `"group"`
- `match.peer.id` : Open ID utilisateur (`ou_xxx`) ou ID de groupe (`oc_xxx`)

Voir [Obtenir les ID de groupe/utilisateur](#obtenir-les-id-de-groupeutilisateur) pour des conseils de recherche.

---

## Référence de configuration

Configuration complète : [Configuration du Gateway](/gateway/configuration)

Options clés :

| Paramètre                                         | Description                       | Par défaut       |
| ------------------------------------------------- | --------------------------------- | ---------------- |
| `channels.feishu.enabled`                         | Activer/désactiver le canal       | `true`           |
| `channels.feishu.domain`                          | Domaine API (`feishu` ou `lark`)  | `feishu`         |
| `channels.feishu.connectionMode`                  | Mode de transport des événements  | `websocket`      |
| `channels.feishu.verificationToken`               | Requis pour le mode webhook       | -                |
| `channels.feishu.webhookPath`                     | Chemin de route webhook           | `/feishu/events` |
| `channels.feishu.webhookHost`                     | Hôte d'écoute webhook             | `127.0.0.1`      |
| `channels.feishu.webhookPort`                     | Port d'écoute webhook             | `3000`           |
| `channels.feishu.accounts.<id>.appId`             | App ID                            | -                |
| `channels.feishu.accounts.<id>.appSecret`         | App Secret                        | -                |
| `channels.feishu.accounts.<id>.domain`            | Domaine API par compte            | `feishu`         |
| `channels.feishu.dmPolicy`                        | Politique de MP                   | `pairing`        |
| `channels.feishu.allowFrom`                       | Liste autorisée MP (open_id)      | -                |
| `channels.feishu.groupPolicy`                     | Politique de groupe               | `open`           |
| `channels.feishu.groupAllowFrom`                  | Liste autorisée de groupe         | -                |
| `channels.feishu.groups.<chat_id>.requireMention` | Exiger @mention                   | `true`           |
| `channels.feishu.groups.<chat_id>.enabled`        | Activer le groupe                 | `true`           |
| `channels.feishu.textChunkLimit`                  | Taille des morceaux de message    | `2000`           |
| `channels.feishu.mediaMaxMb`                      | Limité de taille des médias       | `30`             |
| `channels.feishu.streaming`                       | Activer la sortie carte streaming | `true`           |
| `channels.feishu.blockStreaming`                   | Activer le streaming par blocs    | `true`           |

---

## Référence dmPolicy

| Valeur        | Comportement                                                              |
| ------------- | ------------------------------------------------------------------------- |
| `"pairing"`   | **Par défaut.** Les utilisateurs inconnus reçoivent un code d'appairage ; doit être approuvé |
| `"allowlist"` | Seuls les utilisateurs dans `allowFrom` peuvent discuter                   |
| `"open"`      | Autoriser tous les utilisateurs (nécessite `"*"` dans allowFrom)           |
| `"disabled"`  | Désactiver les MP                                                          |

---

## Types de messages pris en charge

### Réception

- Texte
- Texte riche (post)
- Images
- Fichiers
- Audio
- Vidéo
- Autocollants

### Envoi

- Texte
- Images
- Fichiers
- Audio
- Texte riche (prise en charge partielle)
