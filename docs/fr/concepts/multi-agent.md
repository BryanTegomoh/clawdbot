---
summary: "Routage multi-agent : agents isolés, comptes de canaux et bindings"
title: "Routage multi-agent"
read_when: "Vous voulez plusieurs agents isolés (espaces de travail + authentification) dans un seul processus Gateway."
status: activé
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/multi-agent.md
  workflow: manual
---

# Routage multi-agent

Objectif : plusieurs agents _isolés_ (espace de travail séparé + `agentDir` + sessions), plus plusieurs comptes de canaux (par ex. deux WhatsApp) dans un seul Gateway en cours d'exécution. L'entrant est routé vers un agent via les bindings.

## Qu'est-ce qu'« un agent » ?

Un **agent** est un cerveau entièrement délimité avec ses propres :

- **Espace de travail** (fichiers, AGENTS.md/SOUL.md/USER.md, notes locales, règles de personnalité).
- **Répertoire d'état** (`agentDir`) pour les profils d'authentification, le registre de modèles et la configuration par agent.
- **Magasin de sessions** (historique de chat + état de routage) sous `~/.openclaw/agents/<agentId>/sessions`.

Les profils d'authentification sont **par agent**. Chaque agent lit depuis son propre :

```text
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Les identifiants de l'agent principal ne sont **pas** partagés automatiquement. Ne réutilisez jamais `agentDir` entre les agents (cela cause des collisions d'authentification/session). Si vous voulez partager des identifiants, copiez `auth-profiles.json` dans le `agentDir` de l'autre agent.

Les skills sont par agent via le dossier `skills/` de chaque espace de travail, avec les skills partagés disponibles depuis `~/.openclaw/skills`. Voir [Skills : par agent vs partagés](/tools/skills#per-agent-vs-shared-skills).

Le Gateway peut héberger **un agent** (par défaut) ou **plusieurs agents** côte à côte.

**Note sur l'espace de travail :** l'espace de travail de chaque agent est le **cwd par défaut**, pas un bac à sable strict. Les chemins relatifs se résolvent dans l'espace de travail, mais les chemins absolus peuvent atteindre d'autres emplacements sur l'hôte sauf si l'isolation en bac à sable est activée. Voir [Isolation en bac à sable](/gateway/sandboxing).

## Chemins (carte rapide)

- Configuration : `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`)
- Répertoire d'état : `~/.openclaw` (ou `OPENCLAW_STATE_DIR`)
- Espace de travail : `~/.openclaw/workspace` (ou `~/.openclaw/workspace-<agentId>`)
- Répertoire agent : `~/.openclaw/agents/<agentId>/agent` (ou `agents.list[].agentDir`)
- Sessions : `~/.openclaw/agents/<agentId>/sessions`

### Mode agent unique (par défaut)

Si vous ne faites rien, OpenClaw exécute un seul agent :

- `agentId` est par défaut **`main`**.
- Les sessions sont indexées comme `agent:main:<mainKey>`.
- L'espace de travail est par défaut `~/.openclaw/workspace` (ou `~/.openclaw/workspace-<profile>` quand `OPENCLAW_PROFILE` est défini).
- L'état est par défaut `~/.openclaw/agents/main/agent`.

## Assistant agent

Utilisez l'assistant agent pour ajouter un nouvel agent isolé :

```bash
openclaw agents add work
```

Puis ajoutez les `bindings` (ou laissez l'assistant le faire) pour router les messages entrants.

Vérifiez avec :

```bash
openclaw agents list --bindings
```

## Démarrage rapide

<Steps>
  <Step title="Créer chaque espace de travail d'agent">

Utilisez l'assistant ou créez les espaces de travail manuellement :

```bash
openclaw agents add coding
openclaw agents add social
```

Chaque agent obtient son propre espace de travail avec `SOUL.md`, `AGENTS.md` et optionnellement `USER.md`, plus un `agentDir` dédié et un magasin de sessions sous `~/.openclaw/agents/<agentId>`.

  </Step>

  <Step title="Créer les comptes de canaux">

Créez un compte par agent sur vos canaux préférés :

- Discord : un bot par agent, activer le Message Content Intent, copier chaque token.
- Telegram : un bot par agent via BotFather, copier chaque token.
- WhatsApp : lier chaque numéro de téléphone par compte.

```bash
openclaw channels login --channel whatsapp --account work
```

Voir les guides des canaux : [Discord](/channels/discord), [Telegram](/channels/telegram), [WhatsApp](/channels/whatsapp).

  </Step>

  <Step title="Ajouter agents, comptes et bindings">

Ajoutez les agents sous `agents.list`, les comptes de canaux sous `channels.<channel>.accounts`, et connectez-les avec les `bindings` (exemples ci-dessous).

  </Step>

  <Step title="Redémarrer et vérifier">

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

  </Step>
</Steps>

## Plusieurs agents = plusieurs personnes, plusieurs personnalités

Avec **plusieurs agents**, chaque `agentId` devient une **personnalité entièrement isolée** :

- **Différents numéros de téléphone/comptes** (par `accountId` de canal).
- **Différentes personnalités** (fichiers d'espace de travail par agent comme `AGENTS.md` et `SOUL.md`).
- **Authentification + sessions séparées** (pas de communication croisée sauf si explicitement activée).

Cela permet à **plusieurs personnes** de partager un serveur Gateway tout en gardant leurs « cerveaux » IA et données isolés.

## Un numéro WhatsApp, plusieurs personnes (séparation DM)

Vous pouvez router **différents DM WhatsApp** vers différents agents tout en restant sur **un seul compte WhatsApp**. Faites correspondre l'E.164 de l'expéditeur (comme `+15551234567`) avec `peer.kind: "direct"`. Les réponses proviennent toujours du même numéro WhatsApp (pas d'identité d'expéditeur par agent).

Détail important : les conversations directes se replient vers la **clé de session principale** de l'agent, donc une véritable isolation nécessite **un agent par personne**.

Exemple :

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

Notes :

- Le contrôle d'accès DM est **global par compte WhatsApp** (appairage/liste blanche), pas par agent.
- Pour les groupes partagés, liez le groupe à un agent ou utilisez les [Groupes de diffusion](/channels/broadcast-groups).

## Règles de routage (comment les messages choisissent un agent)

Les bindings sont **déterministes** et le **plus spécifique gagne** :

1. Correspondance `peer` (identifiant exact DM/groupe/canal)
2. Correspondance `parentPeer` (héritage de thread)
3. `guildId + rôles` (routage par rôle Discord)
4. `guildId` (Discord)
5. `teamId` (Slack)
6. Correspondance `accountId` pour un canal
7. Correspondance au niveau canal (`accountId: "*"`)
8. Repli vers l'agent par défaut (`agents.list[].default`, sinon première entrée de la liste, par défaut : `main`)

Si plusieurs bindings correspondent au même niveau, le premier dans l'ordre de configuration gagne.
Si un binding définit plusieurs champs de correspondance (par exemple `peer` + `guildId`), tous les champs spécifiés sont requis (sémantique `ET`).

Detail important concernant la portee des comptes :

- Un binding qui omet `accountId` ne correspond qu'au compte par défaut.
- Utilisez `accountId: "*"` pour un repli a l'echelle du canal sur tous les comptes.
- Si vous ajoutez ensuite le même binding pour le même agent avec un identifiant de compte explicite, OpenClaw met a niveau le binding existant (canal uniquement) vers un binding a portee de compte au lieu de le dupliquer.

## Comptes multiples / numéros de téléphone

Les canaux qui supportent **plusieurs comptes** (par ex. WhatsApp) utilisent `accountId` pour identifier chaque connexion. Chaque `accountId` peut être routé vers un agent différent, pour qu'un serveur puisse héberger plusieurs numéros de téléphone sans mélanger les sessions.

## Concepts

- `agentId` : un « cerveau » (espace de travail, authentification par agent, magasin de sessions par agent).
- `accountId` : une instance de compte de canal (par ex. compte WhatsApp `"personal"` vs `"biz"`).
- `binding` : route les messages entrants vers un `agentId` par `(channel, accountId, peer)` et optionnellement les identifiants de guild/équipe.
- Les conversations directes se replient vers `agent:<agentId>:<mainKey>` (« main » par agent ; `session.mainKey`).

## Exemples par plateforme

### Bots Discord par agent

Chaque compte de bot Discord correspond à un `accountId` unique. Liez chaque compte à un agent et gardez les listes blanches par bot.

```json5
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```

Notes :

- Invitez chaque bot dans la guild et activez le Message Content Intent.
- Les tokens résident dans `channels.discord.accounts.<id>.token` (le compte par défaut peut utiliser `DISCORD_BOT_TOKEN`).

### Bots Telegram par agent

```json5
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```

Notes :

- Créez un bot par agent avec BotFather et copiez chaque token.
- Les tokens résident dans `channels.telegram.accounts.<id>.botToken` (le compte par défaut peut utiliser `TELEGRAM_BOT_TOKEN`).

### Numéros WhatsApp par agent

Liez chaque compte avant de démarrer le Gateway :

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5) :

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Routage déterministe : la première correspondance gagne (plus spécifique en premier).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Surcharge optionnelle par pair (exemple : envoyer un groupe spécifique à l'agent work).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Désactivé par défaut : la messagerie agent-à-agent doit être explicitement activée + en liste blanche.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Surcharge optionnelle. Défaut : ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Surcharge optionnelle. Défaut : ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

## Exemple : chat quotidien WhatsApp + travail approfondi Telegram

Séparer par canal : router WhatsApp vers un agent rapide du quotidien et Telegram vers un agent Opus.

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

Notes :

- Si vous avez plusieurs comptes pour un canal, ajoutez `accountId` au binding (par exemple `{ channel: "whatsapp", accountId: "personal" }`).
- Pour router un seul DM/groupe vers Opus tout en gardant le reste sur chat, ajoutez un binding `match.peer` pour ce pair ; les correspondances de pair gagnent toujours sur les règles au niveau canal.

## Exemple : même canal, un pair vers Opus

Garder WhatsApp sur l'agent rapide, mais router un DM vers Opus :

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

Les bindings de pair gagnent toujours, donc gardez-les au-dessus de la règle au niveau canal.

## Agent familial lié à un groupe WhatsApp

Lier un agent familial dédié à un seul groupe WhatsApp, avec déclenchement par mention et une politique d'outils plus stricte :

```json5
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

Notes :

- Les listes allow/deny d'outils concernent les **outils**, pas les skills. Si un skill doit exécuter un binaire, assurez-vous qu'`exec` est autorisé et que le binaire existe dans le bac à sable.
- Pour un déclenchement plus strict, définissez `agents.list[].groupChat.mentionPatterns` et gardez les listes blanches de groupe activées pour le canal.

## Bac à sable par agent et configuration d'outils

À partir de v2026.1.6, chaque agent peut avoir ses propres restrictions de bac à sable et d'outils :

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // Pas de bac à sable pour l'agent personnel
        },
        // Pas de restrictions d'outils - tous les outils disponibles
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Toujours en bac à sable
          scope: "agent",  // Un conteneur par agent
          docker: {
            // Configuration unique optionnelle après création du conteneur
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Outil read uniquement
          deny: ["exec", "write", "edit", "apply_patch"],    // Refuser les autres
        },
      },
    ],
  },
}
```

Note : `setupCommand` réside sous `sandbox.docker` et s'exécute une seule fois à la création du conteneur.
Les surcharges `sandbox.docker.*` par agent sont ignorées quand la portée résolue est `"shared"`.

**Avantages :**

- **Isolation de sécurité** : restreindre les outils pour les agents non fiables
- **Contrôle des ressources** : isoler certains agents en bac à sable tout en gardant les autres sur l'hôte
- **Politiques flexibles** : permissions différentes par agent

Note : `tools.elevated` est **global** et basé sur l'expéditeur ; il n'est pas configurable par agent.
Si vous avez besoin de limités par agent, utilisez `agents.list[].tools` pour refuser `exec`.
Pour le ciblage de groupe, utilisez `agents.list[].groupChat.mentionPatterns` pour que les @mentions correspondent proprement à l'agent prévu.

Voir [Bac à sable et outils multi-agent](/tools/multi-agent-sandbox-tools) pour des exemples détaillés.
