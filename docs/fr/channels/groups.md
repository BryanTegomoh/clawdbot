---
summary: "Comportement du chat de groupe inter-surfaces (WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams/Zalo)"
read_when:
  - Modification du comportement du chat de groupe ou du filtrage par mention
title: "Groupes"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/groups.md
  workflow: manual
---

# Groupes

OpenClaw traite les chats de groupe de manière cohérente entre les surfaces : WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams, Zalo.

## Introduction pour débutants (2 minutes)

OpenClaw "vit" sur vos propres comptes de messagerie. Il n'y a pas d'utilisateur bot WhatsApp séparé.
Si **vous** êtes dans un groupe, OpenClaw peut voir ce groupe et y répondre.

Comportement par défaut :

- Les groupes sont restreints (`groupPolicy: "allowlist"`).
- Les réponses nécessitent une mention sauf si vous désactivez explicitement le filtrage par mention.

Traduction : les expéditeurs en liste autorisée peuvent déclencher OpenClaw en le mentionnant.

> TL;DR
>
> - **Accès aux Messages privés** est contrôlé par `*.allowFrom`.
> - **Accès aux groupes** est contrôlé par `*.groupPolicy` + listes autorisées (`*.groups`, `*.groupAllowFrom`).
> - **Déclenchement des réponses** est contrôlé par le filtrage par mention (`requireMention`, `/activation`).

Flux rapide (ce qui arrive à un message de groupe) :

```
groupPolicy? disabled -> abandonner
groupPolicy? allowlist -> groupe autorisé? non -> abandonner
requireMention? oui -> mentionné? non -> stocker pour le contexte uniquement
sinon -> répondre
```

![Flux des messages de groupe](/images/groups-flow.svg)

Si vous souhaitez...

| Objectif                                              | Configuration                                                       |
| ----------------------------------------------------- | ------------------------------------------------------------------- |
| Autoriser tous les groupes mais répondre uniquement aux @mentions | `groups: { "*": { requireMention: true } }`                        |
| Désactiver toutes les réponses de groupe              | `groupPolicy: "disabled"`                                           |
| Uniquement des groupes spécifiques                    | `groups: { "<group-id>": { ... } }` (pas de clé `"*"`)             |
| Seul vous pouvez déclencher dans les groupes          | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]`         |

## Clés de session

- Les sessions de groupe utilisent des clés de session `agent:<agentId>:<channel>:group:<id>` (les salons/canaux utilisent `agent:<agentId>:<channel>:channel:<id>`).
- Les sujets de forum Telegram ajoutent `:topic:<threadId>` à l'identifiant du groupe pour que chaque sujet ait sa propre session.
- Les conversations directes utilisent la session principale (ou par expéditeur si configuré).
- Les heartbeats sont ignorés pour les sessions de groupe.

## Motif : Messages privés personnels + groupes publics (agent unique)

Oui, cela fonctionne bien si votre trafic "personnel" est en **Messages privés** et votre trafic "public" est en **groupes**.

Pourquoi : en mode agent unique, les Messages privés tombent généralement dans la clé de session **principale** (`agent:main:main`), tandis que les groupes utilisent toujours des clés de session **non principales** (`agent:main:<channel>:group:<id>`). Si vous activez le bac à sable avec `mode: "non-main"`, ces sessions de groupe s'exécutent dans Docker tandis que votre session principale de Messages privés reste sur l'hôte.

Cela vous donne un "cerveau" d'agent (espace de travail + mémoire partagés), mais deux postures d'exécution :

- **Messages privés** : outils complets (hôte)
- **Groupes** : bac à sable + outils restreints (Docker)

> Si vous avez besoin d'espaces de travail/personas réellement séparés ("personnel" et "public" ne doivent jamais se mélanger), utilisez un second agent + bindings. Voir [Routage Multi-Agent](/concepts/multi-agent).

Exemple (Messages privés sur l'hôte, groupes en bac à sable + outils de messagerie uniquement) :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // les groupes/canaux sont non-main -> en bac à sable
        scope: "session", // isolation la plus forte (un conteneur par groupe/canal)
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        // Si allow est non vide, tout le reste est bloqué (deny l'emporte toujours).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

Vous voulez "les groupes ne peuvent voir que le dossier X" au lieu de "pas d'accès à l'hôte" ? Gardez `workspaceAccess: "none"` et montez uniquement les chemins autorisés dans le bac à sable :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPath:containerPath:mode
            "/home/user/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

En lien :

- Clés de configuration et valeurs par défaut : [Configuration du Gateway](/gateway/configuration#agentsdefaultssandbox)
- Déboguer pourquoi un outil est bloqué : [Bac à sable vs Politique d'outils vs Élevé](/gateway/sandbox-vs-tool-policy-vs-elevated)
- Détails des montages bind : [Bac à sable](/gateway/sandboxing#custom-bind-mounts)

## Labels d'affichage

- Les labels de l'interface utilisent `displayName` quand disponible, formatés comme `<channel>:<token>`.
- `#room` est réservé pour les salons/canaux ; les chats de groupe utilisent `g-<slug>` (minuscules, espaces -> `-`, conserver `#@+._-`).

## Politique de groupe

Contrôlez comment les messages de groupe/salon sont gérés par canal :

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // identifiant utilisateur Telegram numérique (l'assistant peut résoudre @username)
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
    },
  },
}
```

| Politique     | Comportement                                                              |
| ------------- | ------------------------------------------------------------------------- |
| `"open"`      | Les groupes contournent les listes autorisées ; le filtrage par mention s'applique toujours. |
| `"disabled"`  | Bloquer entièrement tous les messages de groupe.                          |
| `"allowlist"` | N'autoriser que les groupes/salons correspondant à la liste autorisée configurée. |

Notes :

- `groupPolicy` est distinct du filtrage par mention (qui nécessite les @mentions).
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo : utilisez `groupAllowFrom` (repli : `allowFrom` explicite).
- Les approbations d'appairage DM (entrées du store `*-allowFrom`) s'appliquent uniquement a l'accès DM ; l'autorisation des expéditeurs de groupe reste explicite aux listes autorisées de groupe.
- Discord : la liste autorisée utilisé `channels.discord.guilds.<id>.channels`.
- Slack : la liste autorisée utilisé `channels.slack.channels`.
- Matrix : la liste autorisée utilisé `channels.matrix.groups` (identifiants de salon, alias ou noms). Utilisez `channels.matrix.groupAllowFrom` pour restreindre les expéditeurs ; les listes autorisées `users` par salon sont également prises en charge.
- Les Messages privés de groupe sont contrôlés séparément (`channels.discord.dm.*`, `channels.slack.dm.*`).
- La liste autorisée Telegram peut correspondre aux identifiants utilisateur (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) ou aux noms d'utilisateur (`"@alice"` ou `"alice"`) ; les préfixes sont insensibles à la casse.
- Par défaut `groupPolicy: "allowlist"` ; si votre liste autorisée de groupes est vide, les messages de groupe sont bloqués.
- Sécurité d'exécution : quand un bloc de fournisseur est complètement absent (`channels.<provider>` absent), la politique de groupe revient à un mode sécurisé (généralement `allowlist`) au lieu d'hériter de `channels.defaults.groupPolicy`.

Modèle mental rapide (ordre d'évaluation des messages de groupe) :

1. `groupPolicy` (open/disabled/allowlist)
2. listes autorisées de groupe (`*.groups`, `*.groupAllowFrom`, liste autorisée spécifique au canal)
3. filtrage par mention (`requireMention`, `/activation`)

## Filtrage par mention (par défaut)

Les messages de groupe nécessitent une mention sauf remplacement par groupe. Les valeurs par défaut se trouvent par sous-système sous `*.groups."*"`.

Répondre au message d'un bot compte comme une mention implicite (quand le canal prend en charge les métadonnées de réponse). Cela s'applique à Telegram, WhatsApp, Slack, Discord et Microsoft Teams.

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

Notes :

- `mentionPatterns` sont des regex insensibles à la casse.
- Les surfaces qui fournissent des mentions explicites passent toujours ; les patterns sont un repli.
- Remplacement par agent : `agents.list[].groupChat.mentionPatterns` (utile quand plusieurs agents partagent un groupe).
- Le filtrage par mention n'est appliqué que lorsque la détection de mention est possible (mentions natives ou `mentionPatterns` sont configurés).
- Les valeurs par défaut de Discord se trouvent dans `channels.discord.guilds."*"` (remplaçables par guilde/canal).
- Le contexte d'historique de groupe est encapsulé uniformément entre les canaux et est **en attente uniquement** (messages ignorés à cause du filtrage par mention) ; utilisez `messages.groupChat.historyLimit` pour la valeur par défaut globale et `channels.<channel>.historyLimit` (ou `channels.<channel>.accounts.*.historyLimit`) pour les remplacements. Définissez `0` pour désactiver.

## Restrictions d'outils par groupe/canal (optionnel)

Certaines configurations de canal prennent en charge la restriction des outils disponibles **à l'intérieur d'un groupe/salon/canal spécifique**.

- `tools` : autoriser/refuser des outils pour tout le groupe.
- `toolsBySender` : remplacements par expéditeur au sein du groupe.
  Utilisez des préfixes de clé explicites :
  `id:<senderId>`, `e164:<phone>`, `username:<handle>`, `name:<displayName>`, et le joker `"*"`.
  Les clés héritées sans préfixe sont toujours acceptées et correspondent uniquement à `id:`.

Ordre de résolution (le plus spécifique l'emporte) :

1. correspondance `toolsBySender` du groupe/canal
2. `tools` du groupe/canal
3. correspondance `toolsBySender` par défaut (`"*"`)
4. `tools` par défaut (`"*"`)

Exemple (Telegram) :

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "id:123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

Notes :

- Les restrictions d'outils par groupe/canal s'appliquent en plus de la politique d'outils globale/agent (deny l'emporte toujours).
- Certains canaux utilisent une imbrication différente pour les salons/canaux (par exemple, Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

## Listes autorisées de groupe

Lorsque `channels.whatsapp.groups`, `channels.telegram.groups`, ou `channels.imessage.groups` est configuré, les clés agissent comme une liste autorisée de groupes. Utilisez `"*"` pour autoriser tous les groupes tout en définissant le comportement de mention par défaut.

Intentions courantes (copier/coller) :

1. Désactiver toutes les réponses de groupe

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2. Autoriser uniquement des groupes spécifiques (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false },
      },
    },
  },
}
```

3. Autoriser tous les groupes mais exiger une mention (explicite)

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4. Seul le propriétaire peut déclencher dans les groupes (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## Activation (propriétaire uniquement)

Les propriétaires de groupe peuvent basculer l'activation par groupe :

- `/activation mention`
- `/activation always`

Le propriétaire est déterminé par `channels.whatsapp.allowFrom` (ou l'E.164 propre du bot quand non défini). Envoyez la commande comme message autonome. Les autres surfaces ignorent actuellement `/activation`.

## Champs de contexte

Les charges utiles entrantes de groupe définissent :

- `ChatType=group`
- `GroupSubject` (si connu)
- `GroupMembers` (si connu)
- `WasMentioned` (résultat du filtrage par mention)
- Les sujets de forum Telegram incluent aussi `MessageThreadId` et `IsForum`.

Le prompt système de l'agent inclut une introduction de groupe au premier tour d'une nouvelle session de groupe. Il rappelle au modèle de répondre comme un humain, d'éviter les tableaux Markdown, et d'éviter de taper des séquences littérales `\n`.

## Spécificités iMessage

- Préférez `chat_id:<id>` pour le routage ou les listes autorisées.
- Listez les chats : `imsg chats --limit 20`.
- Les réponses de groupe reviennent toujours au même `chat_id`.

## Spécificités WhatsApp

Voir [Messages de groupe](/channels/group-messages) pour le comportement spécifique à WhatsApp (injection d'historique, détails de la gestion des mentions).
