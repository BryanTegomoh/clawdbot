---
title: Microsoft Teams
description: Connecter OpenClaw à Microsoft Teams via le Bot Framework Azure.
summary: "Statut de prise en charge du bot Microsoft Teams, fonctionnalités et configuration"
read_when:
  - Vous travaillez sur les fonctionnalités du canal MS Teams
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/msteams.md
  workflow: manual
---

# Microsoft Teams (plugin)

> "Abandon all hope, ye who enter here."

Mis à jour : 2026-01-21

Statut : le texte + les pièces jointes MP sont pris en charge ; l'envoi de fichiers dans les canaux/groupes nécessite `sharePointSiteId` + les permissions Graph (voir [Envoi de fichiers dans les discussions de groupe](#envoi-de-fichiers-dans-les-discussions-de-groupe)). Les sondages sont envoyés via Adaptive Cards.

## Plugin requis

Microsoft Teams est livré comme un plugin et n'est pas inclus dans l'installation de base.

**Changement majeur (2026.1.15) :** MS Teams a été retiré du noyau. Si vous l'utilisez, vous devez installer le plugin.

Explication : cela allège les installations de base et permet aux dépendances MS Teams de se mettre à jour indépendamment.

Installation via CLI (registre npm) :

```bash
openclaw plugins install @openclaw/msteams
```

Checkout local (à partir d'un dépôt git) :

```bash
openclaw plugins install ./extensions/msteams
```

Si vous choisissez Teams pendant la configuration/l'onboarding et qu'un checkout git est détecté,
OpenClaw proposera automatiquement le chemin d'installation local.

Détails : [Plugins](/tools/plugin)

## Configuration rapide (débutant)

1. Installez le plugin Microsoft Teams.
2. Créez un **Azure Bot** (App ID + secret client + tenant ID).
3. Configurez OpenClaw avec ces identifiants.
4. Exposez `/api/messages` (port 3978 par défaut) via une URL publique ou un tunnel.
5. Installez le package d'application Teams et démarrez le Gateway.

Configuration minimale :

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

Note : les discussions de groupe sont bloquées par défaut (`channels.msteams.groupPolicy: "allowlist"`). Pour autoriser les réponses de groupe, définissez `channels.msteams.groupAllowFrom` (ou utilisez `groupPolicy: "open"` pour autoriser tout membre, filtré par mention).

## Objectifs

- Parler à OpenClaw via les Messages privés Teams, les discussions de groupe ou les canaux.
- Garder le routage déterministe : les réponses reviennent toujours au canal d'origine.
- Comportement sûr par défaut pour les canaux (mentions requises sauf configuration contraire).

## Écritures de configuration

Par défaut, Microsoft Teams est autorisé à écrire des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`).

Désactivez avec :

```json5
{
  channels: { msteams: { configWrites: false } },
}
```

## Contrôle d'accès (Messages privés + groupes)

**Accès MP**

- Par défaut : `channels.msteams.dmPolicy = "pairing"`. Les expéditeurs inconnus sont ignorés jusqu'à approbation.
- `channels.msteams.allowFrom` devrait utiliser des identifiants d'objets AAD stables.
- Les UPN/noms d'affichage sont mutables ; la correspondance directe est désactivée par défaut et n'est activée qu'avec `channels.msteams.dangerouslyAllowNameMatching: true`.
- L'assistant peut résoudre les noms en identifiants via Microsoft Graph lorsque les identifiants le permettent.

**Accès groupe**

- Par défaut : `channels.msteams.groupPolicy = "allowlist"` (bloqué sauf si vous ajoutez `groupAllowFrom`). Utilisez `channels.defaults.groupPolicy` pour remplacer la valeur par défaut lorsqu'elle n'est pas définie.
- `channels.msteams.groupAllowFrom` contrôle quels expéditeurs peuvent déclencher dans les discussions de groupe/canaux (revient à `channels.msteams.allowFrom`).
- Définissez `groupPolicy: "open"` pour autoriser tout membre (toujours filtré par mention par défaut).
- Pour n'autoriser **aucun canal**, définissez `channels.msteams.groupPolicy: "disabled"`.

Exemple :

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Liste autorisée d'équipes + canaux**

- Délimitez les réponses de groupe/canal en listant les équipes et canaux sous `channels.msteams.teams`.
- Les clés peuvent être des identifiants ou des noms d'équipes ; les clés de canaux peuvent être des identifiants de conversation ou des noms.
- Lorsque `groupPolicy="allowlist"` et qu'une liste autorisée d'équipes est présente, seuls les équipes/canaux listés sont acceptés (filtrage par mention).
- L'assistant de configuration accepte les entrées `Team/Channel` et les enregistre pour vous.
- Au démarrage, OpenClaw résout les noms d'équipes/canaux et de listes autorisées d'utilisateurs en identifiants (lorsque les permissions Graph le permettent) et journalise la correspondance ; les entrées non résolues sont conservées telles quelles.

Exemple :

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```

## Fonctionnement

1. Installez le plugin Microsoft Teams.
2. Créez un **Azure Bot** (App ID + secret + tenant ID).
3. Construisez un **package d'application Teams** qui référence le bot et inclut les permissions RSC ci-dessous.
4. Téléchargez/installez l'application Teams dans une équipe (ou en portée personnelle pour les Messages privés).
5. Configurez `msteams` dans `~/.openclaw/openclaw.json` (ou variables d'environnement) et démarrez le Gateway.
6. Le Gateway écoute le trafic webhook du Bot Framework sur `/api/messages` par défaut.

## Configuration Azure Bot (prérequis)

Avant de configurer OpenClaw, vous devez créer une ressource Azure Bot.

### Étape 1 : Créer un Azure Bot

1. Rendez-vous sur [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2. Remplissez l'onglet **Basics** :

   | Champ              | Valeur                                                    |
   | ------------------ | --------------------------------------------------------- |
   | **Bot handle**     | Le nom de votre bot, par ex., `openclaw-msteams` (doit être unique) |
   | **Subscription**   | Sélectionnez votre abonnement Azure                       |
   | **Resource group** | Créez-en un ou utilisez un existant                       |
   | **Pricing tier**   | **Free** pour le développement/test                       |
   | **Type of App**    | **Single Tenant** (recommandé, voir note ci-dessous)      |
   | **Création type**  | **Create new Microsoft App ID**                           |

> **Avis de dépréciation :** La création de nouveaux bots multi-tenant a été dépréciée après le 2025-07-31. Utilisez **Single Tenant** pour les nouveaux bots.

3. Cliquez sur **Review + create** puis **Create** (attendez environ 1 à 2 minutes)

### Étape 2 : Obtenir les identifiants

1. Rendez-vous sur votre ressource Azure Bot puis **Configuration**
2. Copiez le **Microsoft App ID** : c'est votre `appId`
3. Cliquez sur **Manage Password** puis allez dans l'App Registration
4. Sous **Certificates & secrets** puis **New client secret** puis copiez la **Value** : c'est votre `appPassword`
5. Allez dans **Overview** puis copiez **Directory (tenant) ID** : c'est votre `tenantId`

### Étape 3 : Configurer le point de terminaison de messagerie

1. Dans Azure Bot puis **Configuration**
2. Définissez le **Messaging endpoint** sur votre URL de webhook :
   - Production : `https://your-domain.com/api/messages`
   - Développement local : utilisez un tunnel (voir [Développement local](#développement-local-tunneling) ci-dessous)

### Étape 4 : Activer le canal Teams

1. Dans Azure Bot puis **Channels**
2. Cliquez sur **Microsoft Teams** puis Configuré puis Save
3. Acceptez les conditions d'utilisation

## Développement local (Tunneling)

Teams ne peut pas atteindre `localhost`. Utilisez un tunnel pour le développement local :

**Option A : ngrok**

```bash
ngrok http 3978
# Copiez l'URL https, par ex., https://abc123.ngrok.io
# Définissez le messaging endpoint sur : https://abc123.ngrok.io/api/messages
```

**Option B : Tailscale Funnel**

```bash
tailscale funnel 3978
# Utilisez votre URL Tailscale Funnel comme messaging endpoint
```

## Portail développeur Teams (alternative)

Au lieu de créer manuellement un fichier ZIP de manifeste, vous pouvez utiliser le [Teams Developer Portal](https://dev.teams.microsoft.com/apps) :

1. Cliquez sur **+ New app**
2. Remplissez les informations de base (nom, description, informations développeur)
3. Allez dans **App features** puis **Bot**
4. Sélectionnez **Enter a bot ID manually** et collez votre Azure Bot App ID
5. Cochez les portées : **Personal**, **Team**, **Group Chat**
6. Cliquez sur **Distribute** puis **Download app package**
7. Dans Teams : **Apps** puis **Manage your apps** puis **Upload a custom app** puis sélectionnez le ZIP

C'est souvent plus simple que de modifier manuellement les manifestes JSON.

## Tester le bot

**Option A : Azure Web Chat (vérifier d'abord le webhook)**

1. Dans le portail Azure, votre ressource Azure Bot puis **Test in Web Chat**
2. Envoyez un message : vous devriez voir une réponse
3. Cela confirme que votre point de terminaison webhook fonctionne avant la configuration Teams

**Option B : Teams (après installation de l'application)**

1. Installez l'application Teams (chargement latéral ou catalogue d'organisation)
2. Trouvez le bot dans Teams et envoyez un Message privé
3. Vérifiez les journaux du Gateway pour l'activité entrante

## Configuration (texte seul minimal)

1. **Installez le plugin Microsoft Teams**
   - Depuis npm : `openclaw plugins install @openclaw/msteams`
   - Depuis un checkout local : `openclaw plugins install ./extensions/msteams`

2. **Enregistrement du bot**
   - Créez un Azure Bot (voir ci-dessus) et notez :
     - App ID
     - Secret client (App password)
     - Tenant ID (single-tenant)

3. **Manifeste de l'application Teams**
   - Incluez une entrée `bot` avec `botId = <App ID>`.
   - Portées : `personal`, `team`, `groupChat`.
   - `supportsFiles: true` (requis pour la gestion des fichiers en portée personnelle).
   - Ajoutez les permissions RSC (ci-dessous).
   - Créez des icônes : `outline.png` (32x32) et `color.png` (192x192).
   - Compressez les trois fichiers ensemble : `manifest.json`, `outline.png`, `color.png`.

4. **Configurez OpenClaw**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   Vous pouvez aussi utiliser des variables d'environnement au lieu des clés de configuration :
   - `MSTEAMS_APP_ID`
   - `MSTEAMS_APP_PASSWORD`
   - `MSTEAMS_TENANT_ID`

5. **Point de terminaison du bot**
   - Définissez le Messaging Endpoint de l'Azure Bot sur :
     - `https://<host>:3978/api/messages` (ou le chemin/port de votre choix).

6. **Exécutez le Gateway**
   - Le canal Teams démarre automatiquement lorsque le plugin est installé et que la configuration `msteams` existe avec des identifiants.

## Contexte d'historique

- `channels.msteams.historyLimit` contrôle le nombre de messages récents de canal/groupe inclus dans le prompt.
- Revient à `messages.groupChat.historyLimit`. Définissez `0` pour désactiver (par défaut 50).
- L'historique MP peut être limité avec `channels.msteams.dmHistoryLimit` (tours utilisateur). Remplacements par utilisateur : `channels.msteams.dms["<user_id>"].historyLimit`.

## Permissions RSC Teams actuelles (manifeste)

Voici les **permissions resourceSpecific existantes** dans notre manifeste d'application Teams. Elles s'appliquent uniquement au sein de l'équipe/discussion où l'application est installée.

**Pour les canaux (portée équipe) :**

- `ChannelMessage.Read.Group` (Application) : recevoir tous les messages de canal sans @mention
- `ChannelMessage.Send.Group` (Application)
- `Member.Read.Group` (Application)
- `Owner.Read.Group` (Application)
- `ChannelSettings.Read.Group` (Application)
- `TeamMember.Read.Group` (Application)
- `TeamSettings.Read.Group` (Application)

**Pour les discussions de groupe :**

- `ChatMessage.Read.Chat` (Application) : recevoir tous les messages de discussion de groupe sans @mention

## Exemple de manifeste Teams (expurgé)

Exemple minimal et valide avec les champs requis. Remplacez les identifiants et les URL.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### Précisions sur le manifeste (champs obligatoires)

- `bots[].botId` **doit** correspondre à l'App ID de l'Azure Bot.
- `webApplicationInfo.id` **doit** correspondre à l'App ID de l'Azure Bot.
- `bots[].scopes` doit inclure les surfaces que vous prévoyez d'utiliser (`personal`, `team`, `groupChat`).
- `bots[].supportsFiles: true` est requis pour la gestion des fichiers en portée personnelle.
- `authorization.permissions.resourceSpecific` doit inclure la lecture/envoi de canal si vous souhaitez du trafic de canal.

### Mise à jour d'une application existante

Pour mettre à jour une application Teams déjà installée (par ex., pour ajouter des permissions RSC) :

1. Mettez à jour votre `manifest.json` avec les nouveaux paramètres
2. **Incrémentez le champ `version`** (par ex., `1.0.0` vers `1.1.0`)
3. **Re-compressez** le manifeste avec les icônes (`manifest.json`, `outline.png`, `color.png`)
4. Téléchargez le nouveau zip :
   - **Option A (Teams Admin Center) :** Teams Admin Center, Teams apps, Manage apps, trouvez votre application, Upload new version
   - **Option B (Chargement latéral) :** Dans Teams, Apps, Manage your apps, Upload a custom app
5. **Pour les canaux d'équipe :** réinstallez l'application dans chaque équipe pour que les nouvelles permissions prennent effet
6. **Quittez complètement et relancez Teams** (pas juste fermer la fenêtre) pour vider les métadonnées d'application en cache

## Fonctionnalités : RSC uniquement vs Graph

### Avec **RSC Teams uniquement** (application installée, pas de permissions Graph API)

Fonctionne :

- Lire le contenu **texte** des messages de canal.
- Envoyer du contenu **texte** dans les messages de canal.
- Recevoir les pièces jointes de fichiers en **Messages privés (DM)**.

Ne fonctionne PAS :

- Contenu d'**images ou fichiers** dans les canaux/groupes (la charge utile n'inclut qu'un stub HTML).
- Téléchargement des pièces jointes stockées dans SharePoint/OneDrive.
- Lecture de l'historique des messages (au-delà de l'événement webhook en direct).

### Avec **RSC Teams + permissions d'application Microsoft Graph**

Ajouté :

- Téléchargement des contenus hébergés (images collées dans les messages).
- Téléchargement des pièces jointes stockées dans SharePoint/OneDrive.
- Lecture de l'historique des messages de canal/discussion via Graph.

### RSC vs API Graph

| Fonctionnalité          | Permissions RSC     | API Graph                            |
| ----------------------- | ------------------- | ------------------------------------ |
| **Messages en temps réel** | Oui (via webhook) | Non (polling uniquement)             |
| **Messages historiques** | Non                | Oui (peut interroger l'historique)   |
| **Complexité de configuration** | Manifeste d'application uniquement | Nécessite consentement admin + flux de tokens |
| **Fonctionne hors ligne** | Non (doit être en cours d'exécution) | Oui (interrogation à tout moment) |

**En résumé :** RSC sert à l'écoute en temps réel ; l'API Graph sert à l'accès historique. Pour rattraper les messages manqués hors ligne, vous avez besoin de l'API Graph avec `ChannelMessage.Read.All` (nécessite le consentement admin).

## Média + historique via Graph (requis pour les canaux)

Si vous avez besoin d'images/fichiers dans les **canaux** ou souhaitez récupérer l'**historique des messages**, vous devez activer les permissions Microsoft Graph et accorder le consentement admin.

1. Dans Entra ID (Azure AD) **App Registration**, ajoutez les permissions **Application** Microsoft Graph :
   - `ChannelMessage.Read.All` (pièces jointes de canal + historique)
   - `Chat.Read.All` ou `ChatMessage.Read.All` (discussions de groupe)
2. **Accordez le consentement admin** pour le tenant.
3. Incrémentez la **version du manifeste** de l'application Teams, re-téléchargez et **réinstallez l'application dans Teams**.
4. **Quittez complètement et relancez Teams** pour vider les métadonnées d'application en cache.

**Permission supplémentaire pour les mentions d'utilisateurs :** Les @mentions d'utilisateurs fonctionnent directement pour les utilisateurs dans la conversation. Cependant, si vous souhaitez rechercher dynamiquement et mentionner des utilisateurs qui **ne sont pas dans la conversation actuelle**, ajoutez la permission `User.Read.All` (Application) et accordez le consentement admin.

## Limitations connues

### Timeouts webhook

Teams livre les messages via webhook HTTP. Si le traitement prend trop longtemps (par ex., réponses LLM lentes), vous pouvez voir :

- Des timeouts du Gateway
- Teams renvoyant le message (causant des doublons)
- Des réponses perdues

OpenClaw gère cela en retournant rapidement et en envoyant les réponses de manière proactive, mais des réponses très lentes peuvent toujours causer des problèmes.

### Formatage

Le markdown Teams est plus limité que Slack ou Discord :

- Le formatage de base fonctionne : **gras**, _italique_, `code`, liens
- Le markdown complexe (tableaux, listes imbriquées) peut ne pas s'afficher correctement
- Les Adaptive Cards sont prises en charge pour les sondages et les envois de cartes arbitraires (voir ci-dessous)

## Configuration

Paramètres clés (voir `/gateway/configuration` pour les motifs de canal partagés) :

- `channels.msteams.enabled` : activer/désactiver le canal.
- `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId` : identifiants du bot.
- `channels.msteams.webhook.port` (par défaut `3978`)
- `channels.msteams.webhook.path` (par défaut `/api/messages`)
- `channels.msteams.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing)
- `channels.msteams.allowFrom` : liste autorisée MP (identifiants d'objets AAD recommandés). L'assistant résout les noms en identifiants pendant la configuration lorsque l'accès Graph est disponible.
- `channels.msteams.dangerouslyAllowNameMatching` : bascule d'urgence pour réactiver la correspondance mutable UPN/nom d'affichage.
- `channels.msteams.textChunkLimit` : taille de découpe du texte sortant.
- `channels.msteams.chunkMode` : `length` (par défaut) ou `newline` pour découper sur les lignes vides (limités de paragraphe) avant la découpe par longueur.
- `channels.msteams.mediaAllowHosts` : liste autorisée pour les hôtes de pièces jointes entrantes (par défaut : domaines Microsoft/Teams).
- `channels.msteams.mediaAuthAllowHosts` : liste autorisée pour l'ajout d'en-têtes Authorization lors des tentatives de téléchargement média (par défaut : hôtes Graph + Bot Framework).
- `channels.msteams.requireMention` : nécessiter un @mention dans les canaux/groupes (par défaut true).
- `channels.msteams.replyStyle` : `thread | top-level` (voir [Style de réponse](#style-de-réponse--fils-vs-publications)).
- `channels.msteams.teams.<teamId>.replyStyle` : remplacement par équipe.
- `channels.msteams.teams.<teamId>.requireMention` : remplacement par équipe.
- `channels.msteams.teams.<teamId>.tools` : remplacements de politique d'outils par défaut par équipe (`allow`/`deny`/`alsoAllow`) utilisés lorsqu'un remplacement de canal est absent.
- `channels.msteams.teams.<teamId>.toolsBySender` : remplacements de politique d'outils par expéditeur par défaut par équipe (joker `"*"` pris en charge).
- `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle` : remplacement par canal.
- `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention` : remplacement par canal.
- `channels.msteams.teams.<teamId>.channels.<conversationId>.tools` : remplacements de politique d'outils par canal (`allow`/`deny`/`alsoAllow`).
- `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender` : remplacements de politique d'outils par expéditeur par canal (joker `"*"` pris en charge).
- Les clés `toolsBySender` doivent utiliser des préfixes explicites :
  `id:`, `e164:`, `username:`, `name:` (les clés héritées sans préfixe correspondent uniquement à `id:`).
- `channels.msteams.sharePointSiteId` : identifiant du site SharePoint pour les téléchargements de fichiers dans les discussions de groupe/canaux (voir [Envoi de fichiers dans les discussions de groupe](#envoi-de-fichiers-dans-les-discussions-de-groupe)).

## Routage & sessions

- Les clés de session suivent le format agent standard (voir [/concepts/session](/concepts/session)) :
  - Les Messages privés partagent la session principale (`agent:<agentId>:<mainKey>`).
  - Les messages de canal/groupe utilisent l'identifiant de conversation :
    - `agent:<agentId>:msteams:channel:<conversationId>`
    - `agent:<agentId>:msteams:group:<conversationId>`

## Style de réponse : fils vs publications

Teams a récemment introduit deux styles d'interface de canal sur le même modèle de données sous-jacent :

| Style                       | Description                                                     | `replyStyle` recommandé |
| --------------------------- | --------------------------------------------------------------- | ----------------------- |
| **Publications** (classique) | Les messages apparaissent comme des cartes avec des réponses en fil en dessous | `thread` (par défaut)  |
| **Fils** (style Slack)       | Les messages s'enchaînent linéairement, comme Slack              | `top-level`            |

**Le problème :** L'API Teams n'expose pas quel style d'interface un canal utilisé. Si vous utilisez le mauvais `replyStyle` :

- `thread` dans un canal en style Fils : les réponses apparaissent imbriquées de manière maladroite
- `top-level` dans un canal en style Publications : les réponses apparaissent comme des publications de niveau supérieur séparées au lieu d'être dans le fil

**Solution :** Configurez `replyStyle` par canal en fonction de la configuration du canal :

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

## Pièces jointes & images

**Limitations actuelles :**

- **Messages privés :** Les images et pièces jointes fonctionnent via les API de fichiers du bot Teams.
- **Canaux/groupes :** Les pièces jointes résident dans le stockage M365 (SharePoint/OneDrive). La charge utile du webhook n'inclut qu'un stub HTML, pas les octets du fichier. **Les permissions de l'API Graph sont requises** pour télécharger les pièces jointes de canal.

Sans les permissions Graph, les messages de canal avec des images seront reçus en texte seul (le contenu de l'image n'est pas accessible au bot).
Par défaut, OpenClaw ne télécharge les médias qu'à partir des noms d'hôtes Microsoft/Teams. Remplacez avec `channels.msteams.mediaAllowHosts` (utilisez `["*"]` pour autoriser tout hôte).
Les en-têtes d'autorisation ne sont attachés que pour les hôtes dans `channels.msteams.mediaAuthAllowHosts` (par défaut : hôtes Graph + Bot Framework). Gardez cette liste stricte (évitez les suffixes multi-tenant).

## Envoi de fichiers dans les discussions de groupe

Les bots peuvent envoyer des fichiers dans les Messages privés en utilisant le flux FileConsentCard (intégré). Cependant, **l'envoi de fichiers dans les discussions de groupe/canaux** nécessite une configuration supplémentaire :

| Contexte                    | Comment les fichiers sont envoyés                         | Configuration nécessaire                           |
| --------------------------- | --------------------------------------------------------- | -------------------------------------------------- |
| **Messages privés**         | FileConsentCard, l'utilisateur accepte, le bot télécharge | Fonctionne directement                             |
| **Discussions de groupe/canaux** | Téléchargement vers SharePoint, partage de lien      | Nécessite `sharePointSiteId` + permissions Graph   |
| **Images (tout contexte)**  | Encodées en base64 en ligne                               | Fonctionne directement                             |

### Pourquoi les discussions de groupe nécessitent SharePoint

Les bots n'ont pas de lecteur OneDrive personnel (le point de terminaison de l'API Graph `/me/drive` ne fonctionne pas pour les identités d'application). Pour envoyer des fichiers dans les discussions de groupe/canaux, le bot télécharge vers un **site SharePoint** et crée un lien de partage.

### Configuration

1. **Ajoutez les permissions de l'API Graph** dans Entra ID (Azure AD), App Registration :
   - `Sites.ReadWrite.All` (Application) : télécharger des fichiers vers SharePoint
   - `Chat.Read.All` (Application) : optionnel, activé les liens de partage par utilisateur

2. **Accordez le consentement admin** pour le tenant.

3. **Obtenez l'identifiant de votre site SharePoint :**

   ```bash
   # Via Graph Explorer ou curl avec un token valide :
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # Exemple : pour un site à "contoso.sharepoint.com/sites/BotFiles"
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # La réponse inclut : "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **Configurez OpenClaw :**

   ```json5
   {
     channels: {
       msteams: {
         // ... autre configuration ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2",
       },
     },
   }
   ```

### Comportement de partage

| Permission                              | Comportement de partage                                     |
| --------------------------------------- | ----------------------------------------------------------- |
| `Sites.ReadWrite.All` uniquement        | Lien de partage à l'échelle de l'organisation (toute personne de l'org peut y accéder) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Lien de partage par utilisateur (seuls les membres de la discussion peuvent y accéder) |

Le partage par utilisateur est plus sécurisé car seuls les participants de la discussion peuvent accéder au fichier. Si la permission `Chat.Read.All` est manquante, le bot revient au partage à l'échelle de l'organisation.

### Comportement de repli

| Scénario                                           | Résultat                                              |
| -------------------------------------------------- | ----------------------------------------------------- |
| Discussion de groupe + fichier + `sharePointSiteId` configuré | Téléchargement vers SharePoint, envoi du lien de partage |
| Discussion de groupe + fichier + pas de `sharePointSiteId`    | Tentative de téléchargement OneDrive (peut échouer), envoi texte uniquement |
| Discussion personnelle + fichier                    | Flux FileConsentCard (fonctionne sans SharePoint)      |
| Tout contexte + image                               | Encodée en base64 en ligne (fonctionne sans SharePoint) |

### Emplacement de stockage des fichiers

Les fichiers téléchargés sont stockés dans un dossier `/OpenClawShared/` dans la bibliothèque de documents par défaut du site SharePoint configuré.

## Sondages (Adaptive Cards)

OpenClaw envoie les sondages Teams sous forme d'Adaptive Cards (il n'y a pas d'API native de sondage Teams).

- CLI : `openclaw message poll --channel msteams --target conversation:<id> ...`
- Les votes sont enregistrés par le Gateway dans `~/.openclaw/msteams-polls.json`.
- Le Gateway doit rester en ligne pour enregistrer les votes.
- Les sondages ne publient pas encore automatiquement de résumés de résultats (inspectez le fichier de stockage si nécessaire).

## Adaptive Cards (arbitraires)

Envoyez n'importe quel JSON Adaptive Card aux utilisateurs ou conversations Teams en utilisant l'outil `message` ou la CLI.

Le paramètre `card` accepte un objet JSON Adaptive Card. Lorsque `card` est fourni, le texte du message est optionnel.

**Outil agent :**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{ "type": "TextBlock", "text": "Hello!" }]
  }
}
```

**CLI :**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

Voir la [documentation Adaptive Cards](https://adaptivecards.io/) pour le schéma et les exemples de cartes. Pour les détails sur le format des cibles, voir [Formats de cibles](#formats-de-cibles) ci-dessous.

## Formats de cibles

Les cibles MSTeams utilisent des préfixes pour distinguer les utilisateurs des conversations :

| Type de cible          | Format                           | Exemple                                             |
| ---------------------- | -------------------------------- | --------------------------------------------------- |
| Utilisateur (par ID)   | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`         |
| Utilisateur (par nom)  | `user:<display-name>`            | `user:John Smith` (nécessite l'API Graph)            |
| Groupe/canal           | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`            |
| Groupe/canal (brut)    | `<conversation-id>`              | `19:abc123...@thread.tacv2` (si contient `@thread`) |

**Exemples CLI :**

```bash
# Envoyer à un utilisateur par ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# Envoyer à un utilisateur par nom d'affichage (déclenche une recherche Graph API)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Envoyer à une discussion de groupe ou un canal
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Envoyer une Adaptive Card à une conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**Exemples d'outil agent :**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{ "type": "TextBlock", "text": "Hello" }]
  }
}
```

Note : Sans le préfixe `user:`, les noms sont résolus par défaut en groupe/équipe. Utilisez toujours `user:` pour cibler des personnes par nom d'affichage.

## Messagerie proactive

- Les messages proactifs ne sont possibles **qu'après** qu'un utilisateur a interagi, car nous stockons les références de conversation à ce moment-là.
- Voir `/gateway/configuration` pour `dmPolicy` et le filtrage par liste autorisée.

## Identifiants d'équipe et de canal (piège courant)

Le paramètre de requête `groupId` dans les URL Teams n'est **PAS** l'identifiant d'équipe utilisé pour la configuration. Extrayez plutôt les identifiants depuis le chemin de l'URL :

**URL d'équipe :**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    ID d'équipe (décodez l'URL ici)
```

**URL de canal :**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      ID de canal (décodez l'URL ici)
```

**Pour la configuration :**

- ID d'équipe = segment de chemin après `/team/` (décodé, par ex., `19:Bk4j...@thread.tacv2`)
- ID de canal = segment de chemin après `/channel/` (décodé)
- **Ignorez** le paramètre de requête `groupId`

## Canaux privés

Les bots ont un support limité dans les canaux privés :

| Fonctionnalité                | Canaux standard    | Canaux privés          |
| ----------------------------- | ------------------ | ---------------------- |
| Installation du bot           | Oui                | Limitée                |
| Messages en temps réel (webhook) | Oui             | Peut ne pas fonctionner |
| Permissions RSC               | Oui                | Peut se comporter différemment |
| @mentions                     | Oui                | Si le bot est accessible |
| Historique API Graph          | Oui                | Oui (avec permissions) |

**Solutions de contournement si les canaux privés ne fonctionnent pas :**

1. Utilisez des canaux standard pour les interactions avec le bot
2. Utilisez les Messages privés : les utilisateurs peuvent toujours envoyer des messages directement au bot
3. Utilisez l'API Graph pour l'accès historique (nécessite `ChannelMessage.Read.All`)

## Dépannage

### Problèmes courants

- **Les images ne s'affichent pas dans les canaux :** Permissions Graph ou consentement admin manquant. Réinstallez l'application Teams et quittez/rouvrez complètement Teams.
- **Pas de réponses dans le canal :** les mentions sont requises par défaut ; définissez `channels.msteams.requireMention=false` ou configurez par équipe/canal.
- **Décalage de version (Teams affiche toujours l'ancien manifeste) :** supprimez + réajoutez l'application et quittez complètement Teams pour rafraîchir.
- **401 Unauthorized du webhook :** Attendu lors de tests manuels sans JWT Azure : signifie que le point de terminaison est accessible mais l'authentification a échoué. Utilisez Azure Web Chat pour tester correctement.

### Erreurs de téléchargement du manifeste

- **« Icon file cannot be empty » :** Le manifeste référence des fichiers d'icônes de 0 octet. Créez des icônes PNG valides (32x32 pour `outline.png`, 192x192 pour `color.png`).
- **« webApplicationInfo.Id already in use » :** L'application est toujours installée dans une autre équipe/discussion. Trouvez-la et désinstallez-la d'abord, ou attendez 5 à 10 minutes pour la propagation.
- **« Something went wrong » lors du téléchargement :** Téléchargez via [https://admin.teams.microsoft.com](https://admin.teams.microsoft.com), ouvrez les DevTools du navigateur (F12), onglet Network, et vérifiez le corps de la réponse pour l'erreur réelle.
- **Échec du chargement latéral :** Essayez « Upload an app to your org's app catalog » au lieu de « Upload a custom app » : cela contourne souvent les restrictions de chargement latéral.

### Les permissions RSC ne fonctionnent pas

1. Vérifiez que `webApplicationInfo.id` correspond exactement à l'App ID de votre bot
2. Re-téléchargez l'application et réinstallez-la dans l'équipe/discussion
3. Vérifiez si votre administrateur d'organisation a bloqué les permissions RSC
4. Confirmez que vous utilisez la bonne portée : `ChannelMessage.Read.Group` pour les équipes, `ChatMessage.Read.Chat` pour les discussions de groupe

## Références

- [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) : guide de configuration Azure Bot
- [Teams Developer Portal](https://dev.teams.microsoft.com/apps) : créer/gérer les applications Teams
- [Schéma du manifeste d'application Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
- [Recevoir des messages de canal avec RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
- [Référence des permissions RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
- [Gestion des fichiers du bot Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (canal/groupe nécessite Graph)
- [Messagerie proactive](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)
