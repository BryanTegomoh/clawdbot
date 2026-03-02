---
summary: "Règles de routage par canal (WhatsApp, Telegram, Discord, Slack) et contexte partage"
read_when:
  - Modification du routage des canaux ou du comportement de la boite de reception
title: "Routage des canaux"
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/channel-routing.md"
  workflow: "manual"
---

# Canaux et routage

OpenClaw route les réponses **vers le canal d'ou le message provient**. Le
modèle ne choisit pas de canal ; le routage est déterministe et contrôle par la
configuration de l'hôte.

## Termes clés

- **Canal** : `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
- **AccountId** : instance de compte par canal (quand pris en charge).
- **AgentId** : un espace de travail isole + magasin de sessions ("cerveau").
- **SessionKey** : la clé de compartiment utilisée pour stocker le contexte et contrôler la concurrence.

## Formes de clés de session (exemples)

Les messages directs se replient sur la session **principale** de l'agent :

- `agent:<agentId>:<mainKey>` (défaut : `agent:main:main`)

Les groupes et canaux restent isoles par canal :

- Groupes : `agent:<agentId>:<channel>:group:<id>`
- Canaux/salons : `agent:<agentId>:<channel>:channel:<id>`

Fils de discussion :

- Les fils Slack/Discord ajoutent `:thread:<threadId>` a la clé de base.
- Les sujets de forum Telegram integrent `:topic:<topicId>` dans la clé de groupe.

Exemples :

- `agent:main:telegram:group:-1001234567890:topic:42`
- `agent:main:discord:channel:123456:thread:987654`

## Règles de routage (comment un agent est choisi)

Le routage choisit **un agent** pour chaque message entrant :

1. **Correspondance exacte de pair** (`bindings` avec `peer.kind` + `peer.id`).
2. **Correspondance de pair parent** (héritage de fil).
3. **Correspondance guild + rôles** (Discord) via `guildId` + `rôles`.
4. **Correspondance guild** (Discord) via `guildId`.
5. **Correspondance team** (Slack) via `teamId`.
6. **Correspondance compte** (`accountId` sur le canal).
7. **Correspondance canal** (tout compte sur ce canal, `accountId: "*"`).
8. **Agent par défaut** (`agents.list[].default`, sinon première entrée de la liste, repli sur `main`).

Quand un binding inclut plusieurs champs de correspondance (`peer`, `guildId`, `teamId`, `rôles`), **tous les champs fournis doivent correspondre** pour que ce binding s'applique.

L'agent correspondant déterminé quel espace de travail et magasin de sessions sont utilisés.

## Groupes de diffusion (exécuter plusieurs agents)

Les groupes de diffusion permettent d'exécuter **plusieurs agents** pour le même pair **quand OpenClaw repondrait normalement** (par exemple : dans les groupes WhatsApp, après le filtrage par mention/activation).

Configuration :

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"],
  },
}
```

Voir : [Groupes de diffusion](/channels/broadcast-groups).

## Aperçu de la configuration

- `agents.list` : définitions d'agents nommes (espace de travail, modèle, etc.).
- `bindings` : associé les canaux/comptes/pairs entrants aux agents.

Exemple :

```json5
{
  agents: {
    list: [{ id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }],
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" },
  ],
}
```

## Stockage des sessions

Les magasins de sessions resident dans le répertoire d'état (défaut `~/.openclaw`) :

- `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Les transcriptions JSONL resident a côté du magasin

Vous pouvez surcharger le chemin du magasin via `session.store` et le template `{agentId}`.

## Comportement WebChat

WebChat se connecte a l'**agent sélectionné** et utilise par défaut la session
principale de l'agent. Grace a cela, WebChat vous permet de voir le contexte
inter-canaux pour cet agent en un seul endroit.

## Contexte de réponse

Les réponses entrantes incluent :

- `ReplyToId`, `ReplyToBody`, et `ReplyToSender` quand disponible.
- Le contexte cite est ajouté a `Body` sous forme de bloc `[Replying to ...]`.

Ceci est coherent à travers les canaux.
