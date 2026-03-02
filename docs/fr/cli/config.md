---
summary: "Référence CLI pour `openclaw config` (lire/modifier/supprimer des valeurs de configuration)"
read_when:
  - Vous souhaitez lire ou modifier la configuration de manière non interactive
title: "config"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/config.md
  workflow: manual
---

# `openclaw config`

Utilitaires de configuration : lire/modifier/supprimer des valeurs par chemin. Exécuter sans sous-commande ouvre l'assistant de configuration (identique a `openclaw configuré`).

## Exemples

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```

## Chemins

Les chemins utilisent la notation point ou crochet :

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

Utilisez l'index de la liste d'agents pour cibler un agent spécifique :

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## Valeurs

Les valeurs sont analysées en JSON5 lorsque possible ; sinon elles sont traitées comme des chaines. Utilisez `--strict-json` pour exiger l'analyse JSON5. `--json` reste supporte comme alias historique.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --strict-json
openclaw config set channels.whatsapp.groups '["*"]' --strict-json
```

Redémarrez le Gateway après les modifications.
