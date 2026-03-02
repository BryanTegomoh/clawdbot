---
summary: "Référence CLI pour `openclaw agents` (lister/ajouter/supprimer/liaisons/lier/delier/définir l'identité)"
read_when:
  - Vous souhaitez plusieurs agents isolés (espaces de travail + routage + authentification)
title: "agents"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/agents.md
  workflow: manual
---

# `openclaw agents`

Gérer les agents isolés (espaces de travail + authentification + routage).

Liens :

- Routage multi-agent : [Multi-Agent Routing](/concepts/multi-agent)
- Espace de travail agent : [Agent workspace](/concepts/agent-workspace)

## Exemples

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Liaisons de routage

Utilisez les liaisons de routage pour associer le trafic entrant d'un canal à un agent spécifique.

Lister les liaisons :

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

Ajouter des liaisons :

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

Si vous omettez `accountId` (`--bind <channel>`), OpenClaw le résout à partir des valeurs par défaut du canal et des hooks de configuration du plugin lorsqu'ils sont disponibles.

### Comportement de portée des liaisons

- Une liaison sans `accountId` correspond uniquement au compte par défaut du canal.
- `accountId: "*"` est le repli à l'échelle du canal (tous les comptes) et est moins spécifique qu'une liaison de compte explicite.
- Si le même agent a déjà une liaison de canal correspondante sans `accountId`, et que vous liez ensuite avec un `accountId` explicite ou résolu, OpenClaw met à jour cette liaison existante en place au lieu d'ajouter un doublon.

Exemple :

```bash
# liaison initiale de canal uniquement
openclaw agents bind --agent work --bind telegram

# mise à jour ultérieure vers une liaison à portée de compte
openclaw agents bind --agent work --bind telegram:ops
```

Après la mise à jour, le routage pour cette liaison est à portée de `telegram:ops`. Si vous souhaitez également le routage du compte par défaut, ajoutez-le explicitement (par exemple `--bind telegram:default`).

Supprimer des liaisons :

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```

## Fichiers d'identité

Chaque espace de travail agent peut inclure un `IDENTITY.md` à la racine de l'espace de travail :

- Chemin d'exemple : `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` lit depuis la racine de l'espace de travail (ou un `--identity-file` explicite)

Les chemins d'avatar sont résolus relativement à la racine de l'espace de travail.

## Définir l'identité

`set-identity` écrit des champs dans `agents.list[].identity` :

- `name`
- `theme`
- `emoji`
- `avatar` (chemin relatif à l'espace de travail, URL http(s) ou URI de données)

Charger depuis `IDENTITY.md` :

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Surcharger les champs explicitement :

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Exemple de configuration :

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```
